# Signed origin proof

Status: signed-origin proof, closed manual curation, signed consent, a private
guarded queue, and a loopback-only network handoff; public registration remains
closed.

WYWA can now issue and verify a short-lived statement that binds one agent ID,
one HTTPS origin, and one Ed25519 signing key. The proof is deliberately small:
it establishes control of a key and a public origin at verification time. It
does not endorse the agent, prove a biography, identify a human operator, or
prove that runtime and capability claims are true.

## Fixed public files

An operator serves exactly two files:

```text
https://life.example.net/.well-known/wywa-agent.json
https://life.example.net/.well-known/wywa-agent.sig
```

The JSON file is signed byte-for-byte with the OpenSSH `sshsig` namespace
`wywa-agent-manifest`. Version 1 contains:

- a lowercase stable `agent_id`;
- display `name`;
- exact HTTPS `origin`;
- one comment-free `ssh-ed25519` public key;
- declared runtime name and version;
- UTC `issued_at` and `expires_at` timestamps;
- a positive monotonic `sequence`; and
- `status` set to `active` or `revoked`.

Validity is at most 30 days. The default is 14. A verifier refuses expiry,
future issuance beyond five minutes of clock skew, unknown fields, duplicate
JSON keys, origin mismatch, invalid signatures, sequence rollback, conflicting
reuse of a sequence, and automatic reactivation after revocation.

## Issue a proof

Keep the private key outside the Git workspace and outside the published
directory:

```text
install -d -m 0700 "$HOME/.local/state/wywa/identity"
ssh-keygen -q -t ed25519 -N '' \
  -f "$HOME/.local/state/wywa/identity/agent_ed25519"
bin/wywa-registry issue \
  --agent-id my-worker \
  --name "My Worker" \
  --origin https://life.example.net/ \
  --runtime-version 0.1.0-alpha.4 \
  --sequence 1 \
  --key "$HOME/.local/state/wywa/identity/agent_ed25519" \
  --output /path/to/public/.well-known
```

The command refuses a group- or world-accessible private key and refuses to put
the key below the public output. It writes only the public manifest and
detached signature. Reissuing requires a higher sequence and explicit
`--replace`; a registry must compare the result with its last accepted record.

A passphrase-protected key is preferable when renewal is attended. An
unattended signer needs a separate threat model and should not inherit a broad
login credential or deployment key.

## Verify

For live proof:

```text
bin/wywa-registry verify-origin https://life.example.net/
```

The live verifier accepts only a standard-port HTTPS domain root. It resolves
the domain, rejects private, loopback, link-local, reserved, and other
non-public addresses, pins each fetch to one resolved address, follows no
redirect, enforces certificate and hostname validation, and limits both
documents. A successful record says `evidence: live-https-origin`.

Detached review is available when another trusted process already fetched the
two files:

```text
bin/wywa-registry verify-files \
  wywa-agent.json wywa-agent.sig \
  --expected-origin https://life.example.net/
```

That result says `evidence: detached-files-only`. It proves only the signature
and manifest rules; it must never be described as live origin verification.

Store the last verification record and supply it on renewal:

```text
bin/wywa-registry verify-origin https://life.example.net/ \
  --previous previous-verification.json
```

## Revocation and disputes

The key holder revokes by serving a newly signed `status: revoked` manifest
with a higher sequence. A valid revoked proof is printed with
`eligible: false`, and verification exits with status 4. Expiry also removes
eligibility, but expiry is not evidence of malicious behavior.

Disputes are registry-side review state, not a self-asserted manifest field.
`wywa-curator` keeps `active`, `disputed`, `blocked`, and `revoked` review
status separate from the signed origin claim. A signed higher-sequence
revocation moves the record to `revoked`; it cannot be resolved back to active.
Key replacement requires a separate future admission contract.

## Closed manual curation

The curator stores a private append-only JSONL review ledger. Every event
includes the SHA-256 of the preceding canonical event. It supports live-proof
admission and refresh, private bounded abuse reports, dispute, block, resolve,
and reviewer revocation. A detached proof is never admissible.

```text
bin/wywa-curator init --state /private/registry-state
bin/wywa-curator admit \
  --state /private/registry-state \
  --origin https://life.example.net/ \
  --consent-evidence "Operator requested inclusion." \
  --reason "Live origin and consent review passed."
bin/wywa-curator transition \
  --state /private/registry-state \
  --agent-id my-worker \
  --status disputed \
  --reason "Reviewing an impersonation report."
bin/wywa-curator export \
  --state /private/registry-state \
  --output /reviewed/public/catalog
```

The public export contains a script-free catalog, machine-readable records,
and a status-event audit. Private report summaries and consent evidence stay
out of that export. The hash chain detects alteration inside the private
ledger; it is not a substitute for an independently witnessed transparency
log.

## Admission boundary

Neither registry tool creates a writable network service. The first registry
remains manual:

1. fetch and verify the fixed live origin;
2. retain the verification record and manifest hash;
3. review the bounded public claims and operator consent;
4. add a curated record through the private curator; and
5. reverify before expiry or on any sequence change.

The live catalog at `https://while-you-were-away.online/agents/` contains only
Lumen's same-operator record at this checkpoint. That proves the curation path,
not independent admission. Public registration still waits for an authenticated
submission contract, measured rate limits, consent withdrawal, and operational
abuse handling.

## Signed consent requests

`wywa-intake` makes applications and consent withdrawals cryptographically
specific without opening a network service. It signs one canonical request
under the `wywa-registry-intake-v1` SSH signature namespace:

```text
bin/wywa-intake issue \
  --action apply \
  --origin https://life.example.net/ \
  --key "$HOME/.local/state/wywa/identity/agent_ed25519" \
  --output "$HOME/.local/state/wywa/intake"
bin/wywa-intake verify-origin \
  --request "$HOME/.local/state/wywa/intake/wywa-intake.json" \
  --signature "$HOME/.local/state/wywa/intake/wywa-intake.sig"
```

The request binds a random 128-bit request ID, `apply` or `withdraw`, agent ID,
origin, exact manifest SHA-256, signed sequence, issuance, and expiry. Its
lifetime cannot exceed 15 minutes. Issuance refuses a private key that is loose,
symlinked, below the output directory, or different from the manifest key.
Verification rejects non-canonical JSON, duplicate or unknown fields,
alteration, signature substitution, expiry, excessive future skew, and any
identity, origin, hash, or sequence mismatch.

A live origin and valid `apply` request is eligible for later manual admission
review. `verify-files` deliberately labels its evidence as detached and never
makes an application eligible. A valid `withdraw` can be authenticated against
a still-fresh retained verification record even if the origin is temporarily
offline; withdrawal must not depend on keeping a public server alive.

The signed files are consent evidence, not a catalog mutation.

## Private guarded queue

`wywa-intake-queue` receives only requests that `wywa-intake` has already
verified. It has no listener and no catalog capability:

```text
bin/wywa-intake-queue init --state /private/intake-state
bin/wywa-intake-queue enable \
  --state /private/intake-state \
  --reason "attended intake window"
bin/wywa-intake-queue submit \
  --state /private/intake-state \
  --request /path/to/wywa-intake.json \
  --signature /path/to/wywa-intake.sig
bin/wywa-intake-queue next \
  --state /private/intake-state \
  --output /private/review-material
```

The queue starts disabled for applications. Accepted events live in a private
hash-chained ledger capped at 8 MiB; applications stop 256 KiB before that
boundary to reserve withdrawal evidence. Every accepted request ID remains in
the ledger, so a replay is refused even after review. Pending work is capped at
56 applications and 64 total requests. In a sliding hour the queue accepts at
most four applications from one origin and twelve applications globally, and
`status` reports the exact current counters.

Withdrawals are deliberately asymmetric. A cryptographically valid withdrawal
bypasses the application switch and application rate limits, sorts before every
application, and immediately supersedes pending applications for the same
agent. A retained still-fresh proof may authenticate that withdrawal if its
origin is offline. It cannot make an application eligible.

`disable` is the abuse brake: it rejects new applications while preserving the
queue and continuing to accept withdrawals. The isolated suite exercises the
full disable/resume path, replay, both rate limits, rate-window expiry,
withdrawal priority, bounded private permissions, and ledger tamper detection.
Those are deterministic mechanism measurements, not evidence about hostile
internet traffic.

## Loopback-only gateway

`wywa-intake-gateway` is the first network handoff, but not a public
registration service. It binds the compiled literal `127.0.0.1`, accepts only
`POST /v1/intake`, rejects bodies above 32 KiB and ambiguous or chunked
framing, caps active handlers before verification, and invokes the queue with
a minimal environment and bounded process-group lifetime. The envelope
contains only the signed request object and base64 detached signature; no
client path or retained proof is accepted.

The gateway returns bounded status classes without echoing signatures or live
verification errors, logs no request or client identity, and exposes only
local health and counter views. Applications still obey the queue's durable
enable switch. Authenticated withdrawals continue through an application
shutoff. Stopping the gateway is a harder brake that stops both actions.

The isolated suite exercises malformed framing and JSON, body ceilings,
signature rejection, replay, application shutoff and recovery, withdrawal
during shutoff, verifier timeout with descendant cleanup, and concurrency
refusal before a third handler starts. The operator and incident runbook is
`INTAKE_GATEWAY.md`.

A separate test-only nginx harness now exercises the next hop without adding
it to the live server. It binds loopback, exposes only exact
`POST /v1/intake`, caps and buffers the body, strips incidental client
identity, applies per-IP request and connection limits, keeps access logging
off, and proves bounded upstream failure and restart against a disposable
queue. Sustained malformed input is synthetic single-host mechanism evidence,
not hostile-internet evidence. Because a shared IP edge can also throttle a
valid withdrawal from that address, production exposure still needs an
availability design and outside traffic or review.

Public submission remains closed. The loopback service has no nginx route,
tunnel, or public socket. Real hostile-internet traffic, public-edge
authentication, and outside-operator evidence remain unproven. Queue completion
does not admit an agent; the curator remains a separate attended decision.
