# Security review map

WYWA is alpha software for unattended AI work. Its public dashboard and
portable worker are usable, but the registry intake edge is deliberately not
public. The live gateway binds only to IPv4 loopback, applications are disabled
in the durable queue, and production nginx has no intake route.

This document is a map for reviewers, not a claim that the system is secure.

Every public `main` push and pull request gets a fresh GitHub-hosted rebuild of
the allowlisted standalone tree. The workflow has an empty permissions map,
does not persist a checkout credential, invokes no third-party action, and
checks that the fetched ref resolves to the exact event commit before running
the suites. The host-profile check runs as root only inside the disposable
runner so it can exercise administrator-owned paths; no repository permission
or secret is granted to that job. This makes a review attempt independently
visible without treating CI as a reviewer or a second operator.

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
11. Curator ledger replacement preserves the private directory's exact owner
   and group, and the mutation lock cannot be a symlink or an unsafe shared
   inode.
12. One unattended child cannot grow any one file beyond 16 MiB; durable
   promotion additionally refuses JSONL at that ceiling, diagnostics above
   4 MiB, or a final message above 64 KiB.
13. The host refuses durable checkpoint promotion above 128 MiB of logical
   workspace content or 4,096 paths and rechecks the exact candidate Git tree
   before commit.
14. When the optional root-administered `wywa-volume` profile is used, the
   workspace is its own preallocated fixed-size ext4 mount with a fixed inode
   count. Version-9 receipts record and recheck that live filesystem ceiling.
   An ordinary directory is labeled unbounded by WYWA instead of borrowing the
   post-turn checkpoint claim.
15. Timer-launched workers run as one cgroup-bounded systemd user service:
    memory pressure begins at 70% of installed RAM, the unit is killed at 80%,
    swap is capped at 25% of configured swap, CPU at 200%, and tasks at 256.
    An OOM stops the unit. The attended `wywa-life trial` remains inside the
    caller's existing resource boundary.

The detailed contracts are in
[`REGISTRY.md`](REGISTRY.md) and
[`INTAKE_GATEWAY.md`](INTAKE_GATEWAY.md).

## Known gaps

- The workspace-only filesystem profile is a beta Codex permission surface.
  WYWA pins the reviewed unattended boundary to Codex 0.145.0 rather than
  silently trusting an upgrade. Local-command network is disabled. Apps,
  plugins, hooks, browser/computer control, image generation, subagents,
  approvals, user config, project rules, workspace-local Codex config, and
  session persistence are also disabled or refused explicitly; live hosted web
  search is the only non-shell external tool retained. A live catalog probe
  showed no connector, plugin, browser, image-generation, or subagent tool.
  The surviving local `view_image` tool could not resolve a known public PNG
  outside the disposable workspace, so it honored the deny-read boundary.
  Current version-9 receipts record this launch posture. Every Codex upgrade must
  repeat the filesystem, catalog, image-path, command-egress, and hosted-search
  probes before the version pin moves.
- The aggregate checkpoint gate limits accepted continuity, not live storage.
  The optional `wywa-volume` profile now supplies a real filesystem-level
  block and inode ceiling for a dedicated host, but it requires a root
  administrator and an empty target before bootstrap. The default portable
  directory remains explicitly unbounded by WYWA during the turn.
- The scheduled cgroup defaults protect the host proportionally; they are not
  workload sizing advice and do not apply to a directly invoked attended
  `trial` or `wywa run`. A workload can still fail inside the unit when its
  memory or task demand crosses the configured ceiling.
- The public intake route is closed. One attended 56-second window reached the
  exact split edge from three successful GitHub-hosted jobs while applications
  stayed disabled. It proved public DNS/TLS routing, edge body and method
  refusal, reserved signed withdrawal, replay handling, early rollback, and
  queue drain. It did not retain client addresses, so it does not claim three
  distinct source IPs.
- A second attended 110-second window ran the tagged provider-neutral probe on
  Hugging Face CPU Basic. It repeated the static, method, body, application
  brake, and withdrawal checks, then restored the exact static nginx source
  and drained the queue. The provider record is authenticated rather than
  anonymously public. NAT contention, upstream DDoS controls, sustained
  hostile internet behavior, outside review, and an independent operator
  remain untested.
- Retained withdrawal proofs are operator-staged private state. Their refresh,
  inspection, and removal are now atomic and origin-specific, with live HTTPS
  re-verification, monotonic history, renewal status, and verified-revocation
  retirement. Export creates an identity-signed capsule; import requires an
  independently pinned public key and rejects tampering, expiry, and sequence
  rollback. Lumen's current capsule is encrypted to the owner's existing SSH
  recovery key and held in the off-host owner chat. The timer does not update
  that copy or renew the signed manifest, and a present but stale or malformed
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
tests/test-curator
tests/test-edge-probe
```

The five suites use disposable keys, origins, curator and queue state, ports,
certificates, workers, and nginx configuration. They do not alter production
services or need a real Codex login, domain, or credential.

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
