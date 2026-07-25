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
`SHA-256(origin).json`. Refresh and inspect a reviewed live verification with:

```text
install -d -m 0700 /private/retained
bin/wywa-retained-proof refresh \
  --origin https://life.example.net/ \
  --directory /private/retained
bin/wywa-retained-proof status \
  --origin https://life.example.net/ \
  --directory /private/retained
```

The refresh command invokes the sibling live HTTPS verifier, automatically
uses the retained record as monotonic-sequence history, rejects verifier
output older than five minutes or with less than one day of validity, and
replaces the exact origin-hash file atomically at mode 0600. A fetch,
validation, or short-validity failure preserves the preceding record. A
verified higher-sequence revocation removes the old active record. `status`
returns nonzero when renewal is due, the record is expired or revoked, or it
is absent. `remove` retires only the exact origin-hash file.

Lumen's current operator profile installs
`platform/wywa-retained-proof.service` and
`platform/wywa-retained-proof.timer`. The sandboxed root oneshot has network
access, read-only reviewed executables, and write access only to the retained
directory. It runs twice daily with randomized delay; a verified revocation is
an expected terminal result, while verification failure leaves the unit failed
and preserves the previous record. The timer refreshes the verification record
only. It cannot renew the separately signed origin manifest.

### Off-host recovery capsule

`export` signs the exact canonical retained record with the agent's Ed25519
identity key under the dedicated
`wywa-retained-proof-backup-v1` namespace. It writes one mode-0600 capsule in a
pre-existing mode-0700 directory:

```text
install -d -m 0700 /private/recovery
bin/wywa-retained-proof export \
  --origin https://life.example.net/ \
  --directory /private/retained \
  --key /private/identity/agent_ed25519 \
  --output /private/recovery/retained.capsule.json
```

Move an encrypted copy of that capsule to storage outside the host. Do not put
the plaintext capsule, the identity private key, or a reusable decryption key
in the source repository. Retain the expected Ed25519 public key through a
separate trusted channel or reviewed source checkpoint; a key copied only from
the capsule would be circular evidence.

After whole-host loss, decrypt the capsule into a private regular file, create
the new retained directory, and import against that independently pinned key:

```text
install -d -m 0700 /private/retained
chmod 0600 /private/recovery/retained.capsule.json /private/expected-agent.pub
bin/wywa-retained-proof import \
  --origin https://life.example.net/ \
  --directory /private/retained \
  --capsule /private/recovery/retained.capsule.json \
  --expected-key /private/expected-agent.pub
```

Import verifies the record digest, signature, exact origin and public key,
active status, remaining validity, and sequence monotonicity before one atomic
write. Repeating the same capsule is idempotent. An older sequence, a
same-sequence conflict, a different key, altered content, or an expired record
fails closed. The capsule preserves evidence that the origin was live at its
recorded `verified_at`; it does not claim the origin is live during recovery
and cannot extend the signed manifest's expiry.

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
6. Refresh or inspect retained proof before re-enabling applications:
   `bin/wywa-retained-proof refresh --origin ORIGIN --directory DIRECTORY`.
7. Re-enable applications only in an attended window after the failure is
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

## First bounded public window

On 2026-07-25, a 56-second attended window installed the matching split edge
only after an independent eight-minute automatic rollback was armed. Three
GitHub-hosted jobs reached the public origin. All three observed static HTTPS
200, apply GET 405, malformed apply 400, oversized apply 413, and a signed
application 503 while applications remained disabled. One signed withdrawal
returned 202 and its two replays returned 409.

The application worker recorded three malformed rejections and three disabled
signed requests, with zero accepted applications. The withdrawal worker
recorded one accepted request, two conflicts, and three retained-proof
selections. Neither worker overloaded, rate-limited, or timed out. The site was
restored early and byte for byte, nginx no longer referenced either loopback
port, and the attended withdrawal was finished without curation. Public
intake is closed.

The exact workflow is public on branch `edge-probe-2026-07-25`, commit
`da1d71770223a9c096b7302c1bacb4732379bd1a`; Actions run
[`30147397453`](https://github.com/IlluminationAi/while-you-were-away/actions/runs/30147397453)
contains the three successful hosted jobs. The source distribution includes
the attended-only nginx template and traffic-zone file. This proves one
provider's outside routing, not distinct source IPs, unrelated networks, NAT
contention, sustained hostile behavior, or internet readiness.

## Provider-neutral outside probe

`wywa-edge-probe` carries the exact bounded assertion path without depending
on a CI vendor or third-party Python package. Give it two short-lived signed
envelopes only after the application brake is closed and the automatic edge
rollback is armed:

```text
export WYWA_EDGE_ORIGIN=https://life.example.net
export WYWA_EDGE_PROVIDER=independent-executor
export WYWA_EDGE_APPLY_ENVELOPE_B64=...
export WYWA_EDGE_WITHDRAW_ENVELOPE_B64=...
bin/wywa-edge-probe
```

The client refuses redirects, non-HTTPS origins, swapped actions, mismatched
origin or manifest state, expired envelopes, unexpected application-brake
responses, and responses over 64 KiB. It checks static service, wrong-method,
malformed-body, oversized-body, signed-application, and signed-withdrawal
behavior, then emits one path-free JSON record without the envelopes.

`provider` and `execution_id` are labels, not remote attestation. Preserve the
provider's independently fetchable job or run record beside the JSON result.
The envelopes are one-use public consent evidence, never a reusable signing
credential. The probe does not arm rollback, open nginx, drain the queue, or
curate a record; those remain separate attended operator actions.

## Second bounded public window

On 2026-07-25, Hugging Face CPU Basic job
`6a6469e77ef3c08464968294` ran the exact alpha.6 `wywa-edge-probe` after the
application brake and an independent eight-minute rollback were verified.
The provider record names the tagged script URL, CPU Basic flavor, 180-second
timeout, two secret variable names, completed status, and two-second runtime.
The client emitted:

```json
{"claims":{"client_identity_retained_by_probe":false,"provider_identity_proven_by_output_alone":false},"execution_id":"6a6469e77ef3c08464968294","origin":"https://while-you-were-away.online/","provider":"hugging-face","result":"pass","schema_version":1,"statuses":{"apply":503,"malformed":400,"oversized":413,"static":200,"withdraw":202,"wrong_method":405}}
```

The attended edge was open from 07:46:05 to 07:47:55 UTC. Applications
remained disabled, one withdrawal entered the reserved lane, and no curator
capability was reachable. The operator restored static nginx SHA-256
`b1c00faf885542f43b0934b862db876821d00e10a882164ac68ca415e5180900`,
removed the zones file, observed public POST return 405, and finished the
withdrawal as queue event 16 without curation. The final queue is disabled and
empty with audit head
`c51d0cf38f18cd678e7597a1055a6f50a0b2aa26be1cc7557e29223a09a62ff5`.

The monitoring URL is
`https://huggingface.co/jobs/pakkonen/6a6469e77ef3c08464968294`, but it
redirects anonymous visitors to login. The provider account can re-inspect the
record; the path-free client result alone cannot attest where it ran. This is
therefore real second-provider execution with weaker public provenance than
the GitHub Actions run, not a second operator, outside review, hostile traffic,
or permission to leave intake open.
