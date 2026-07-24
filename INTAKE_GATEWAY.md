# Private intake gateway

Status: two action-pinned loopback workers are operational. There is no public
submission route, production reverse proxy, or automatic catalog mutation.

`wywa-intake-gateway` accepts one versioned JSON envelope at
`POST /v1/intake` on `127.0.0.1`. The live profile runs an application worker
on port 8742 and a withdrawal worker on port 8743. Each process is pinned to
its one expected signed action. The envelope contains only the canonical
signed request object and its detached signature:

```json
{
  "schema_version": 1,
  "request": {
    "schema_version": 1,
    "request_id": "32 lowercase hexadecimal characters",
    "action": "apply or withdraw",
    "agent_id": "signed agent ID",
    "origin": "https://life.example.net/",
    "manifest_sha256": "64 lowercase hexadecimal characters",
    "sequence": 1,
    "issued_at": "UTC timestamp",
    "expires_at": "UTC timestamp"
  },
  "signature_b64": "base64 OpenSSH signature"
}
```

The gateway accepts no client-supplied path, retained verification file,
curator command, or free-text catalog field. It canonicalizes the request,
writes the request and signature into a private temporary directory, and
invokes `wywa-intake-queue submit` with a minimal environment. The process
group is killed on verification timeout. A successful HTTP response means only
that the signed request entered the private review queue.

## Retained proof for an offline origin

The withdrawal-only worker may receive
`--retained-verification-dir /private/retained`. The directory must be a
private nonsymlink directory, and each proof is a private regular file named
`SHA-256(origin).json`. For example, a reviewed live verification for
`https://life.example.net/` is installed as:

```text
origin=https://life.example.net/
proof_name=$(printf '%s' "$origin" | sha256sum | cut -d' ' -f1)
install -d -m 0700 /private/retained
install -m 0600 reviewed-verification.json \
  "/private/retained/$proof_name.json"
```

The client still sends only its signed request and signature. The worker hashes
the origin inside that signed request and either selects the exact server-side
file or performs live verification when no file exists. The intake verifier
then checks the request signature, agent, origin, manifest hash, sequence, and
proof freshness. A selected proof that is stale, malformed, loose, symlinked,
or mismatched fails closed. This mode is refused unless the process is pinned
with `--expected-action withdraw`; it can never make an application eligible.

## Fixed controls

- listener: IPv4 loopback `127.0.0.1` only;
- body: one non-chunked `application/json` body, at most 32 KiB;
- signed request and signature: at most 16 KiB each;
- concurrency: four by default, refused before another handler thread starts;
- read timeout: five seconds;
- live-origin handoff timeout: 50 seconds;
- routes: `POST /v1/intake`, plus local `GET/HEAD /healthz` and `/status`;
- logs: no request or client-identity log; `/status` retains bounded counters;
- authority: private queue append only, with no curator capability.

The application brake remains the queue's durable switch. Disable it without
stopping authenticated withdrawals:

```text
bin/wywa-intake-queue disable \
  --state /private/intake-state \
  --reason "emergency intake shutoff"
```

Reopen applications only for an attended window:

```text
bin/wywa-intake-queue enable \
  --state /private/intake-state \
  --reason "attended intake window"
```

Stopping only the application worker is the harder application network brake;
the withdrawal worker remains available. Stop both workers only when the HTTP
parser, verification chain, or host itself is suspect.

## Run and inspect

Keep the gateway and its sibling `wywa-intake-queue`, `wywa-intake`, and
`wywa-registry` executables in one root-controlled directory:

```text
bin/wywa-intake-gateway serve \
  --state /private/intake-state \
  --port 8742 \
  --max-concurrency 4 \
  --expected-action apply \
  --status-file /run/wywa-intake-apply-gateway/status.json
bin/wywa-intake-gateway serve \
  --state /private/intake-state \
  --port 8743 \
  --max-concurrency 1 \
  --expected-action withdraw \
  --retained-verification-dir /private/retained \
  --status-file /run/wywa-intake-withdraw-gateway/status.json
curl --noproxy '*' http://127.0.0.1:8742/healthz
curl --noproxy '*' http://127.0.0.1:8743/healthz
```

The bounded health and status documents publish both the action pin and the
SHA-256 of the executing gateway source. They expose whether a retained-proof
directory is configured and a count of selections, but not its path or proof
content. This makes an upgrade, rollback, and retained fallback observable on
each lane. A client cannot choose or override either pin.

Before any start or restart, verify that the queue state is a mode-0700,
nonsymlink directory and that the listener address is not configurable beyond
the compiled loopback literal. Afterward, use `ss` to confirm there is no
wildcard or public bind. Do not add this service to nginx, a tunnel, a public
socket unit, or a load balancer.

## Response contract

- `202`: the signed request was appended to the private queue;
- `400` or `415`: malformed framing or envelope;
- `409`: the signed request ID was already seen;
- `422`: signature, identity, origin, freshness, or proof verification failed;
- `429`: queue rate, pending-work, or ledger capacity was reached;
- `503`: applications are disabled or gateway concurrency is full;
- `504`: live verification exceeded the bounded handoff time.

Responses never echo the signature or a verifier's network error detail.

## Incident and recovery

1. Disable queue applications. Withdrawals continue.
2. Capture `/status`, queue `status`, the unit state, and the loopback socket
   binding. Do not copy signed bodies into an ordinary log.
3. If the application parser or process is in doubt, stop the application
   worker and leave the separately pinned withdrawal worker running. Stop both
   only if their shared verification chain or host is suspect. Preserve the
   private queue ledger; do not truncate it.
4. Verify the ledger hash chain and exact installed executables against a
   reviewed source checkpoint.
5. Restart with applications still disabled. Exercise `/healthz`, one
   malformed request, and one authenticated withdrawal or disposable signed
   fixture.
6. Re-enable applications only in an attended window after the failure is
   understood.

`next` and `finish` remain attended queue operations. Finishing a request does
not call `wywa-curator`; admission, dispute, block, and revocation remain a
separate manual authority boundary.

## Isolated reverse-proxy harness

`platform/intake-edge-harness.nginx.conf.in` is test-only. It binds a random
IPv4 loopback TLS port with an ephemeral certificate, not 80 or 443, and
proxies to separate disposable action-pinned workers sharing one queue.
`tests/test-intake-edge` validates the real nginx and TLS parsers and proves:

- only exact `POST /v1/intake/apply` and `/v1/intake/withdraw` reach their
  respective workers;
- each worker rejects a signed action sent through the wrong route;
- query variants, chunked framing, content encodings, wrong media types, and
  bodies above 32 KiB stop at the edge;
- independent per-IP request and connection ceilings reject sustained
  application load before it consumes withdrawal capacity;
- two slow applications can fill the application connection budget while a
  valid signed withdrawal from the same address still enters its reserved
  worker;
- client identity, cookies, authorization, referrer, and user-agent headers
  are not forwarded;
- access and rate-limit logging remain off; and
- upstream loss returns one bounded 503 response, while a fresh gateway on the
  same socket restores accepted intake without changing the queue; and
- a staged executable upgrade changes both workers' published source hashes,
  a one-worker failure leaves signed withdrawal available, a disabled queue
  enforces withdrawal-only emergency posture, and an atomic rollback restores
  both prior hashes without merging the signed-action lanes; and
- removing the withdrawal origin from the live test verifier still permits a
  signed withdrawal through the server-held retained proof, while applications
  continue to require live origin evidence.

This is mechanism evidence from one host and one source address. It does not
test public routing, distributed addresses, NAT contention, provider DDoS
controls, or a real hostile internet. The split closes the specific
shared-throttle and one-worker lifecycle flaws; public exposure still requires
outside review or bounded real-traffic evidence.
