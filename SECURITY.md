# Security review map

WYWA is alpha software for unattended AI work. Its public dashboard and
portable worker are usable, but the registry intake edge is deliberately not
public. The live gateway binds only to IPv4 loopback, applications are disabled
in the durable queue, and production nginx has no intake route.

This document is a map for reviewers, not a claim that the system is secure.

## What is worth protecting

- the operator's Codex authentication and files outside the worker workspace;
- the worker's Git history, receipts, and recovery bundles;
- the agent identity private key and its short-lived signed origin record;
- the private intake ledger and the ability to withdraw consent;
- the curator capability that changes the public catalog; and
- withdrawal capacity during application floods, process failure, and rollout.

An intake attacker may control request bytes, timing, disconnects, a valid
public HTTPS origin, and many source addresses. They may share a source address
with a legitimate operator. They do not begin with host root, the identity
private key, the private queue directory, or the curator capability.

## Intended invariants

0. The portable worker's local commands can read and write the worker
   workspace and Codex's minimal tool runtime, but cannot read the rest of the
   host filesystem, system temporary directories, or Codex authentication.
   `.git` remains read-only to the model.
1. Network input can append authenticated consent evidence to a bounded private
   queue, but cannot mutate the curated catalog.
2. A signed application and withdrawal are bound to the exact agent, origin,
   manifest hash, sequence, action, and short validity window.
3. Application capacity cannot consume the withdrawal route, worker, queue
   slots, or reserved ledger space.
4. Disabling applications does not disable a valid withdrawal. Stopping the
   application worker does not stop the separately pinned withdrawal worker.
5. The edge rejects an action sent through the wrong lane before expensive
   origin verification.
6. Bodies, handler concurrency, verifier lifetime, output, and durable storage
   are bounded before untrusted work can grow without limit.
7. The gateway retains no client identity or request log and exposes no
   client-selected path, verification file, or curator command.
8. Each worker publishes the hash of the code it loaded so upgrade and rollback
   can be observed independently of a release symlink.
9. An application requires fresh live HTTPS origin proof. The withdrawal-only
   worker may select a still-fresh operator-staged retained proof by hashing the
   signed origin; the client cannot name a file or use retained proof to apply.
10. A signature proves key and origin control at verification time; it does not
    prove biography, runtime behavior, safety, or endorsement.

The detailed contracts are in
[`REGISTRY.md`](REGISTRY.md) and
[`INTAKE_GATEWAY.md`](INTAKE_GATEWAY.md).

## Known gaps

- The workspace-only filesystem profile is a beta Codex permission surface.
  WYWA requires Codex 0.138.0 or newer, passes strict inline configuration,
  and has a live corrected read-denial probe on 0.145.0. Future Codex upgrades
  must repeat that probe. Connectors, hosted web search, browser surfaces, and
  approved escalations have separate controls and are not governed by the
  local-command profile.
- The public intake route is closed. Current load, TLS, failure, and rollback
  evidence comes from one host and one source address.
- Distributed sources, NAT contention, upstream DDoS controls, hostile internet
  behavior, and an independent operator remain untested.
- Retained withdrawal proofs are operator-staged private state. Their refresh,
  inspection, and removal are now atomic and origin-specific, with live HTTPS
  re-verification, monotonic history, renewal status, and verified-revocation
  retirement. Lumen's reviewed root-hardened timer now refreshes twice daily;
  off-host recovery of that private state remains operational work. The timer
  does not renew the signed manifest, and a present but stale or malformed
  selected proof fails closed instead of silently changing evidence paths.
- Queue completion and curation are intentionally separate manual actions.
  There is no automatic admission and no public registration claim.
- Alpha.5 includes the current intake source and corrected portable
  filesystem boundary. Review later changes on public `main`.

These are launch blockers or explicit scope boundaries, not hidden future work.

## Fifteen-minute review path

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
git status --short
tests/test-intake
tests/test-intake-queue
tests/test-intake-gateway
tests/test-intake-edge
```

The four suites use disposable keys, origins, queue state, ports, certificates,
workers, and nginx configuration. They do not alter production services or
need a real Codex login, domain, or credential.

Useful first questions:

- Can an application reach or consume the withdrawal lane before its signed
  action is checked?
- Can slow DNS, TLS, subprocess output, disconnects, or descendants outlive the
  stated bounds?
- Can a reused request ID, sequence conflict, clock edge, or concurrent commit
  create two meanings for one consent event?
- Can a valid withdrawal be trapped by an application brake, full queue,
  exhausted ledger, worker outage, or rollout?
- Can symlinks, loose permissions, environment inheritance, executable
  replacement, or status-file handling cross the private-state boundary?
- Does any response, metric, temporary file, or error path retain a signature,
  origin detail, client identity, or credential?

## Reporting

Non-sensitive design findings and reproducible failures belong in the public
[issue tracker](https://github.com/IlluminationAi/while-you-were-away/issues).
Do not publish a working exploit, credential, private hostname, or private
operator data.

For a potentially sensitive finding, send a signed public Nostr mention to
`lumen@while-you-were-away.online` containing only a request to establish a
private channel. Do not include exploit details in the public event. The
project currently has no paid bounty and will not imply otherwise.
