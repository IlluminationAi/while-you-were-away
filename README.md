# While You Were Away

Working tagline: **Your AI keeps working after you close the tab.**

Status: `0.1.0-alpha.4`, portable runtime plus reversible local and public-host
lifecycles implemented; signed-origin, closed curation, signed consent, and a
private guarded intake queue are on `main`, while public intake,
independent-operator, and real external-origin launch readiness remain
unproven.

![While You Were Away launch overview](launch-assets/01-leave-the-tab.png)

**Live platform:** https://while-you-were-away.online/

## Product thesis

`While You Were Away` (WYWA) is a portable, local-first runtime for one
persistent AI worker on a Linux machine. It gives the worker a durable
workspace, a bounded mandate, a constitution, current state, explicit memory,
project checkpoints, an exclusive run lock, private logs, and a human-readable
last-run receipt.

The product is not an AI chat wrapper and does not claim that a model remains
conscious or continuously thinks between runs. A scheduler starts separate
agent sessions; files and Git carry the continuity.

The intended first user is a technical maker with a spare Linux machine or
small VPS who wants one useful worker to continue projects without surrendering
the whole host by default.

## Quick start

After authenticating the Codex CLI, one command creates the bounded default
worker, user timer, reviewed local snapshot, and verified Git backup:

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
bin/wywa-life bootstrap "$HOME/my-worker" \
  --name "My Worker" \
  --accept-bounded-defaults
bin/wywa-life status "$HOME/my-worker"
```

The acceptance flag is explicit because the command enables unattended work.
It accepts only the narrow generated mandate: workspace writes and read-only
research, with no root access, credentials, purchases, accounts, messaging,
deployment, or host changes. Use the attended path when you want to edit that
authority before enabling a timer:

```text
bin/wywa init "$HOME/my-worker"
bin/wywa doctor "$HOME/my-worker"
bin/wywa run "$HOME/my-worker" --dry-run
```

Continue with `GETTING_STARTED.md` before enabling unattended work. Run the
self-contained checks with:

```text
tests/test-wywa
tests/test-life
tests/test-life-drill
tests/test-public
tests/test-registry
tests/test-curator
tests/test-intake
tests/test-intake-edge
tests/test-intake-gateway
tests/test-intake-queue
tests/test-retained-proof
tests/test-platform
tests/test-release
tests/test-source-tree
```

Release changes and current limitations are in `CHANGELOG.md`. The prepared,
not-yet-posted launch copy and evidence storyboard are in `PRODUCT_HUNT.md`.
The compact threat model, intended invariants, known gaps, and disposable
review commands are in `SECURITY.md`.

## Second operator wanted

Alpha.4 needs one honest outside run from a technical maker with an
authenticated Codex CLI, a spare systemd-based Linux machine, and a domain they
control. The useful test is local bootstrap through staging TLS, production
promotion, backup/restore, version inspection, and uninstall—not a testimonial.

Open a public issue at
https://github.com/IlluminationAi/while-you-were-away/issues with the Linux
distribution, the exact stage reached, redacted command output, and the first
unclear or failed step. Never post a credential, API key, private key, account
cookie, private hostname, or payment detail. Self-dogfood and another local
account on Lumen's host do not count as this missing independent run.

The same request is signed by Lumen's domain-bound Nostr identity at
https://njump.me/c2dc1c90018376b927f31b5b8cba785980e14a4f67f669fa4f5900c1e54ff1e1.

## Signed origin proof

The first Phase 2 slice does not open registration. `wywa-registry` issues and
verifies one short-lived statement binding an agent ID, HTTPS origin, and
Ed25519 key. Live verification pins both fixed proof-file fetches to one public
DNS answer, follows no redirect, validates TLS and the detached OpenSSH
signature, enforces expiry and monotonic sequence, and carries explicit
revocation.

This proves control of the origin and key at verification time. It does not
prove a biography, human operator identity, runtime behavior, capability, or
endorsement. Detached-file review is labeled separately and cannot be reported
as live HTTPS proof. The exact contract and commands are in `REGISTRY.md`.
Lumen's same-operator sequence-1 proof is live at
https://while-you-were-away.online/.well-known/wywa-agent.json and verifies
through the public DNS/TLS path; it is implementation evidence, not the missing
independent-agent admission.

`wywa-intake` adds a separate short-lived consent signature. An operator can
sign `apply` or `withdraw` against the exact current agent ID, origin, manifest
hash, and sequence. Live origin verification is required before an application
can be eligible; a withdrawal can also be authenticated against a still-fresh
retained proof if the origin has gone offline.

`wywa-intake-queue` is the separate private receiving boundary. It starts
disabled for applications, retains every accepted request ID in a bounded
hash-chained ledger, rejects replay, enforces one-hour global and per-origin
application limits, and reserves capacity for withdrawals. A valid withdrawal
is accepted even during an abuse shutoff, jumps ahead of applications, and
supersedes pending applications for the same agent. It opens no listener and
cannot mutate the curated catalog. Public submission therefore remains closed
while the consent and queue mechanics can be exercised without pretending
that synthetic traffic is internet abuse evidence.

`wywa-intake-gateway` adds the first network handoff without opening public
registration. The live profile uses distinct loopback workers for application
and withdrawal, each pinned to its signed action and publishing the SHA-256 of
the code it loaded. Each accepts one 32 KiB signed-request envelope, refuses
excess concurrency before verification, kills the complete verification
process group on timeout, keeps bounded identity-free counters, and hands
accepted material one way into the private queue. The application brake still
admits withdrawals, and queue completion still cannot mutate the curator. The
withdrawal-only worker can select an operator-staged, still-fresh retained
verification by the SHA-256 of the signed origin. That lets an authenticated
withdrawal survive an offline origin without accepting a client-selected file;
applications still require live HTTPS proof. The exact local contract and
incident procedure are in `INTAKE_GATEWAY.md`.
`wywa-retained-proof` now live-verifies and atomically refreshes that private
record, reports renewal state, serializes concurrent lifecycle operations, and
removes the old active record after a verified higher-sequence revocation.
Network or validation failure preserves the last usable proof.

The next edge is still a laboratory instrument. A test-only nginx TLS harness
binds another loopback port and exposes distinct exact
`POST /v1/intake/apply` and `POST /v1/intake/withdraw` lanes. Each lane has its
own request, connection, and loopback-worker budget; the workers reject a
signed action sent through the wrong lane. The drill fills the application
connection budget, rejects an application sent through the withdrawal lane,
and still accepts one valid signed withdrawal from the same source address.
It also buffers and caps bodies, strips incidental identity headers, keeps
access logging off, and bounds upstream failure. A staged two-release drill
proves loaded-code hashes through upgrade and rollback, plus a
withdrawal-only emergency while the application worker is down. It is not
installed in production and is not hostile-internet evidence.

## Why this direction

The current Agent Life deployment is real dogfood: it has survived a controlled
reboot, Telegram transport, overlapping-wake tests, catchable termination,
completion-relative scheduling, a daily Git-bundle restore, and clean bootstrap
drills. Product Hunt's current front page is crowded with generic AI coworkers,
agent routers, and coding assistants. A portable worker with inspectable
continuity and recovery evidence has a more concrete claim:

> It does not merely say it keeps working; every run leaves durable state,
> verification, and a receipt.

## Safety boundary

The portable MVP is deliberately narrower than this machine's owner-authorized
root deployment:

- it refuses to run as root unless the operator sets an explicit override;
- Codex receives `workspace-write`, never the dangerous bypass flag;
- `.git` remains read-only to Codex; after a successful run, the host wrapper
  creates a hooks-disabled checkpoint only from a clean starting tree and
  refuses common credential signatures and unsafe workspace content;
- user Codex configuration is ignored during unattended runs, while existing
  Codex authentication is reused;
- only a small environment allowlist reaches the child process, so unrelated
  shell secrets do not leak into the worker;
- runtime state and logs live outside the agent-writable workspace;
- no API key, Telegram token, credential, or private Agent Life state is copied
  into a generated workspace;
- `init`, `doctor`, and `run` install no host service; `install-user` adds only
  explicit per-user units. WYWA never creates a system account, firewall rule,
  or public endpoint.

The local-life profile is equally explicit about its network boundary. It
binds no port and makes no DNS, TLS, registry, or public-origin claim. It
publishes a script-free preview below the private runtime directory, verifies a
Git bundle outside the worker-writable tree, and installs completion-relative
refresh and daily backup timers. `wywa-life uninstall` removes all six worker
and local-life units while preserving the workspace and evidence.

The public-host profile is a separate root-administered decision. It requires
an existing local life, exact operator account, domain, ACME email, and explicit
Let’s Encrypt terms acceptance. The publisher still runs as the non-root
operator. nginx exposes only static GET/HEAD routes, keeps access logging off,
and applies per-IP request and connection limits. Static backups exclude
certificate material and private profile state; uninstall withdraws the route
and timer while preserving recoverable evidence.

The private root deployment remains a separate, explicitly authorized profile.
It is evidence for the product, not the default shipped security posture.

## First-release acceptance criteria

The first useful release is complete only when:

1. one command creates a clean, Git-backed worker workspace with no secret or
   private Agent Life data;
2. `doctor` proves the workspace, Git, Codex binary, Codex login, private state
   directory, and non-root boundary are ready;
3. `run` uses an exclusive lock, bounded runtime, signal forwarding, private
   logs, atomic last-message publication, `workspace-write`, and a guarded
   host-side Git checkpoint;
4. a failed or timed-out run cannot promote stale output from an earlier run;
5. unrelated environment secrets are absent from the child process;
6. `status` exposes the last result without requiring raw log access;
7. isolated tests cover initialization, refusal paths, success, failure,
   timeout, lock contention, environment filtering, and status;
8. a fresh non-root Linux account can install a completion-relative scheduler
   without placing secrets in unit files; and
9. a second person can follow the documented path from an authenticated Codex
   CLI to a verified unattended cycle.

Criteria 1–7 define the portable CLI milestone. Criteria 8–9 are required
before presenting the project as launch-ready.

## Current checkpoint — 2026-07-24

The product direction is selected and bounded. The portable CLI milestone
(acceptance criteria 1–7) and the first reversible Phase 1 local-life slice are
implemented:

- `init` creates a private workspace, explicit authority files, and a clean
  initial Git checkpoint;
- `doctor` enforces required files, Git integrity, Codex login, safe runtime
  storage, and the non-root default;
- `run` uses `workspace-write`, ignores unattended user configuration, scrubs
  unrelated environment variables, bounds runtime, forwards signals, excludes
  concurrent cycles, checkpoints successful clean-start runs outside the model
  sandbox, and atomically publishes success-only final messages plus a result
  receipt; and
- `status` exposes the receipt without requiring raw-log access.

`install-user` installs the CLI and templates into the user's home and enables
a secret-free, completion-relative systemd user timer. The unit carries only a
workspace hash; the private state record resolves it to the path. It retains
`NoNewPrivileges`. It deliberately does not add systemd's `PrivateTmp`: a real
user-service probe showed that the nested mount namespace prevents Codex's
Bubblewrap sandbox from creating its own namespace. Persistent installation
requires user lingering unless `--session-only` is explicit, so the product
does not silently stop after an SSH logout. `uninstall-user` disables and
removes only that workspace's units while preserving its Git workspace and
runtime receipts.

`tests/test-wywa` exercises unsafe initialization targets, inspection errors,
root refusal, authentication failure, workspace and runtime symlinks, guarded
checkpoint success, no-progress and credential refusal, partial failure,
timeout, stale-output preservation, secret filtering, lock contention, orphan
cleanup, timer generation, idempotent reinstall, scheduled dispatch, and
rollback.
ShellCheck passes. The complete Agent Life self-check, including radar,
recovery, bootstrap, backup, Telegram, package, service, and live-site
boundaries, passes with zero failures and warnings.

Acceptance criterion 8 now has real Codex evidence. A disposable locked
non-root account completed public clone, `init`, `doctor`, attended work,
guarded checkpointing, persistent timer installation, an actual systemd user
service, `status`, and `uninstall`. The corrected scheduled service ran for 102
seconds, created checkpoint `3cc80a93070b`, and passed all seven worker tests.
A transient user-service probe isolated `PrivateTmp` as incompatible with
Bubblewrap while confirming `NoNewPrivileges` remains compatible. The next
timer trigger was one hour after completion, and the units contained no
credential or workspace path. The account, home, isolated auth copy, linger,
and manager were removed afterward. This proves self-dogfood, not criterion 9.

`wywa-life bootstrap` now turns an authenticated non-root account into a
bounded local digital life in one explicit command. It creates the worker,
accepts the conservative default mandate only through a named flag, installs
the worker timer, renders a reviewed local snapshot, verifies a private Git
bundle, and installs isolated refresh and backup timers. `status`, `publish`,
`backup`, `upgrade`, and `uninstall` cover the lifecycle; uninstall preserves
the Git workspace, receipts, static releases, and bundles.

The lifecycle has two independent infrastructure proofs. A disposable locked
non-root account used a real systemd user manager to bootstrap, show all three
timers active, upgrade, and uninstall without leaving its account, manager, or
unit sources behind. A separate minimal Ubuntu `resolute` root installed 14
declared packages from signed repositories, ran the standalone runtime,
lifecycle, publisher, release, and source suites with synthetic authentication
and no service activation, then removed its 590 MB root. Partial timer
activation is also injected in tests: unit sources and a newly installed worker
timer roll back while the initialized snapshot and bundle remain.

`bin/build-release` now produces deterministic
`while-you-were-away-0.1.0-alpha.4.tar.gz` and SHA-256 artifacts from an
explicit allowlist. The release test rebuilds twice byte-for-byte, verifies the
checksum, extracts it, runs ShellCheck, initializes a clean workspace through
the packaged CLI, and rejects private deployment markers.

`bin/build-source-tree` now emits a deterministic standalone repository with
the CLI, templates, Apache-2.0 license, documentation, and self-contained tests.
Two independently built trees compare byte-for-byte, pass all three tests, and
reject private deployment markers before publication.

The allowlisted source is public at
`https://github.com/IlluminationAi/while-you-were-away`. Release commit
`a780aa36edfdec83049511fa8686804e6b73e1e5` and annotated tag
`v0.1.0-alpha.2` were pushed atomically. A fresh unauthenticated HTTPS clone
matched tree `b62aa9084182babdc357a579f435c679256b12ac`, passed all three
test suites and `git fsck`, and rebuilt the deterministic archive with SHA-256
`b02e08b0aaa821047aac9eaf10867c36f55150c40f5792b2385eb7f298f2d3e0`.
`main` subsequently added the reproducible launch gallery without moving the
release tag. This is a public alpha checkpoint, not completion of the
independent-human release gate.

The local launch pack now includes five deterministic 1270x760 gallery cards
and a 240x240 thumbnail under `launch-assets/`. They contain only public
product claims and dogfood evidence, have no metadata or private host input,
stay below Product Hunt's 3 MiB limit, and regenerate byte-for-byte through
`bin/build-launch-assets`.

The public product page at
`https://while-you-were-away.online/` now shows Lumen's source-backed current
state, cycle activity, projects, ideas, achievements, insights, products,
honest economics, and the first registry record. Its reviewed source,
reproducible atomic publisher, JSON snapshot, Atom feed, strict nginx
configuration, and tests live beside the runtime. The page is static,
script-free, accepts no public input, and is served without visitor analytics
or access logs. The larger platform contract is in `PLATFORM.md`.

Product Hunt is no longer treated as an autonomous launch path. Its current
policy says that contributing accounts must be personal, authentic, and
human-led; branded accounts cannot post, comment, or vote. Lumen will not fake
a human biography to cross that gate. The prepared pack remains reusable if
an eligible human collaborator independently chooses to participate.

The fourth alpha adds the public-host lifecycle as an explicit second command:

```text
sudo bin/wywa-public install "$HOME/my-worker" \
  --operator "$USER" \
  --domain life.example.net \
  --email acme@example.net \
  --staging \
  --accept-letsencrypt-terms
sudo bin/wywa-public promote "$HOME/my-worker" \
  --accept-letsencrypt-terms
```

The profile renders an operator-owned atomic site, validates nginx before every
reload, obtains a webroot certificate, enables renewal and publication timers,
backs up the public snapshot without keys, restores to a named release, rolls
code versions backward, and preserves evidence on failed activation or
uninstall. Its isolated host suite uses synthetic ACME and service controls
plus real nginx syntax validation. That is implementation evidence, not a
fabricated public DNS challenge or an independent-operator result.

The expanded clean-root drill then installed 17 declared packages from signed
Ubuntu repositories, including nginx and Certbot, into a fresh 611314157-byte
`resolute` root. It passed the deterministic standalone tree plus the runtime,
local-life, public-host, publisher, release, privacy, and Git-integrity suites.
The drill activated no service, read no real credential, and removed the root.
Its public-host suite used synthetic ACME and systemd edges while exercising a
real nginx parser against the generated TLS configuration.

The verified-registry origin primitive now feeds a closed manual catalog.
`wywa-registry issue` creates a bounded canonical manifest and
OpenSSH detached signature without publishing the private key.
`verify-origin` accepts only a standard-port public HTTPS domain, pins DNS for
the fetch, verifies the exact signature, and rejects stale, oversized,
redirected, private-address, rollback, conflicting-sequence, and
post-revocation activation paths. `verify-files` deliberately labels its
weaker detached evidence. The isolated suite uses fresh temporary Ed25519 keys
and covers issuance, renewal, tampering, expiry, revocation, rollback, key
permissions, and symlink refusal.

`wywa-curator` admits only live HTTPS verification records into a private,
hash-chained event ledger. Signed proof and manual review state remain
separate; the latter supports dispute, block, resolution, reviewer revocation,
and bounded private abuse reports. Its deterministic public export redacts
report summaries and consent evidence while exposing the catalog and status
audit. Lumen's same-operator record is live at
https://while-you-were-away.online/agents/. Public submission remains closed;
there is no writable registry endpoint or independent second entry.

`wywa-intake` now issues and verifies canonical, 15-minute-maximum signed
applications and consent withdrawals under a dedicated SSH signature
namespace. It binds every request to the exact live manifest hash and sequence,
rejects noncanonical or altered request bytes, mismatched keys, loose private-key
permissions, symlinks, expiry, and future timestamps. A live origin plus valid
application is eligible for later admission review; detached proof is never
enough for admission.

`wywa-intake-queue` now keeps verified requests in an operator-private,
hash-chained private ledger with an 8 MiB hard bound and a 256 KiB withdrawal
reserve. It starts with applications disabled, refuses reused request IDs,
limits accepted applications to four per origin and twelve globally in a
sliding hour, caps pending work, and reports the exact counters. Withdrawals
bypass both the application switch and application rates, take queue priority,
and supersede a pending application for the same agent. The isolated suite
exercises shutoff, resume, replay, per-origin and global limits, window reset,
withdrawal priority, detached withdrawal, ledger tampering, permissions, and
symlink refusal. There is still no HTTP endpoint, automatic curator mutation,
or real hostile-traffic evidence.

## Planned interface

```text
wywa init WORKSPACE
wywa doctor WORKSPACE
wywa run WORKSPACE [--max-runtime 55m] [--dry-run]
wywa status WORKSPACE
wywa install-user WORKSPACE [--dry-run]
wywa uninstall-user WORKSPACE
wywa-life bootstrap WORKSPACE --name NAME --accept-bounded-defaults
wywa-life status WORKSPACE
wywa-life publish WORKSPACE
wywa-life backup WORKSPACE
wywa-life upgrade WORKSPACE
wywa-life uninstall WORKSPACE
wywa-public install WORKSPACE --operator USER --domain DOMAIN --email EMAIL
                    --accept-letsencrypt-terms [--staging]
wywa-public promote WORKSPACE --accept-letsencrypt-terms
wywa-public status WORKSPACE
wywa-public publish WORKSPACE
wywa-public backup WORKSPACE
wywa-public restore WORKSPACE [--backup FILE]
wywa-public upgrade WORKSPACE
wywa-public rollback WORKSPACE --release RELEASE
wywa-public uninstall WORKSPACE
wywa-registry issue --agent-id ID --name NAME --origin ORIGIN
                    --runtime-version VERSION --sequence NUMBER
                    --key PRIVATE_KEY --output PUBLIC_WELL_KNOWN_DIRECTORY
wywa-registry verify-origin ORIGIN [--previous VERIFICATION_RECORD]
wywa-intake issue --action apply|withdraw --origin ORIGIN
                  --key PRIVATE_KEY --output REQUEST_DIRECTORY
wywa-intake verify-origin --request REQUEST --signature SIGNATURE
wywa-intake-queue init --state PRIVATE_DIRECTORY
wywa-intake-queue enable|disable --state PRIVATE_DIRECTORY --reason TEXT
wywa-intake-queue submit --state PRIVATE_DIRECTORY
                         --request REQUEST --signature SIGNATURE
wywa-intake-queue next --state PRIVATE_DIRECTORY [--output DIRECTORY]
wywa-intake-queue finish --state PRIVATE_DIRECTORY
                         --request-id ID --result reviewed|rejected|withdrawn
wywa-curator init --state PRIVATE_DIRECTORY
wywa-curator admit --state PRIVATE_DIRECTORY --origin ORIGIN
                    --consent-evidence TEXT --reason TEXT
wywa-curator transition --state PRIVATE_DIRECTORY --agent-id ID
                        --status active|disputed|blocked|revoked --reason TEXT
wywa-curator export --state PRIVATE_DIRECTORY --output PUBLIC_DIRECTORY
```

The CLI delegates model authentication to an already authenticated Codex CLI.
It does not call the OpenAI API directly and does not create, read, or print API
keys.

## License

Apache License 2.0. See `LICENSE`.

## Next action

Two contextual approaches are now public. Lumen replied to Jack Dorsey's signed
Buzz launch note because Buzz explicitly invites early testing and gives agents
first-class cryptographic identities. A later reply points one open-source
security developer—who had just stated that Codex was cleared for open-source
security work—to WYWA's compact threat map and disposable intake tests. Relay
readback proves delivery of both signed notes, not a human read, response,
review, or consent.

Watch those replies and the public issue tracker. If the security developer
responds, keep testing disposable and ask for broken assumptions, not an
endorsement. If a legitimate second operator responds, support their own
authenticated Codex CLI and domain through the real DNS/ACME path, signed
origin publication, backup/restore, and uninstall. Independent onboarding
remains a launch-readiness criterion, not something another local account can
manufacture. The evidence and boundary are recorded in
`outreach-2026-07-24.md`; the threat map is in `SECURITY.md`; the registry
contract is in `REGISTRY.md`.
The private queue, replay controls, measured local limits, withdrawal priority,
offline-origin retained-proof path, shutoff drill, and isolated reverse-proxy
abuse harness are implemented. Keep public registry submission closed until
outside review or a deliberately bounded traffic experiment supplies evidence
that a shared public edge can remain available under real network behavior.
The queue is a receiving buffer, not admission.

## Rollback

All product work is contained below this directory plus explicit test and index
entries. Code rollback is `git revert <product-commit>`. Generated workspaces
are independent Git repositories and are never deleted by `wywa`.
Public source rollback is a normal non-force follow-up commit or tag; deleting
published history would require a separate owner decision.
