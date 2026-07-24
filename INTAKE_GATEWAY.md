# Private intake gateway

Status: loopback-only operational boundary. There is no public submission
route, reverse proxy, or automatic catalog mutation.

`wywa-intake-gateway` accepts one versioned JSON envelope at
`POST /v1/intake` on `127.0.0.1`. The envelope contains only the canonical
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

Stopping the gateway is the harder network brake. It also stops live
withdrawals, so prefer the application switch unless the HTTP parser,
verification chain, or host itself is suspect.

## Run and inspect

Keep the gateway and its sibling `wywa-intake-queue`, `wywa-intake`, and
`wywa-registry` executables in one root-controlled directory:

```text
bin/wywa-intake-gateway serve \
  --state /private/intake-state \
  --port 8742 \
  --max-concurrency 4 \
  --status-file /run/wywa-intake-gateway/status.json
curl --noproxy '*' http://127.0.0.1:8742/healthz
curl --noproxy '*' http://127.0.0.1:8742/status
```

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
3. If parser or process integrity is in doubt, stop the gateway. Preserve the
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
IPv4 loopback port, not 80 or 443, and proxies to a disposable gateway and
queue. `tests/test-intake-edge` validates the real nginx parser and proves:

- only exact `POST /v1/intake` reaches the gateway;
- query variants, chunked framing, content encodings, wrong media types, and
  bodies above 32 KiB stop at the edge;
- per-IP request and two-connection ceilings reject sustained malformed load
  before it consumes gateway verification slots;
- client identity, cookies, authorization, referrer, and user-agent headers
  are not forwarded;
- access and rate-limit logging remain off; and
- upstream loss returns one bounded 503 response, while a fresh gateway on the
  same socket restores accepted intake without changing the queue.

This is mechanism evidence from one host and one source address. It does not
test TLS, distributed addresses, NAT contention, provider DDoS controls, or a
real hostile internet. The shared edge limit can temporarily throttle a valid
withdrawal from the same address as abusive traffic; the queue's durable
withdrawal privilege begins only after a request crosses the proxy. That
availability gap is a launch boundary, not a footnote to erase.
