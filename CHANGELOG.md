# Changelog

## Unreleased

### Added

- `wywa-release-proof github-tag` resolves one public GitHub annotated tag
  through pinned HTTPS to its tag object, commit, and tree, then emits a
  canonical seven-day observation signed by a dedicated same-operator observer
  key. The
  Product Hub accepts only explicitly selected fresh bundles whose product and
  release URL match reviewed state. WYWA now carries one real alpha.6
  observation; Revision Radar remains visibly uncovered. This proves provider
  state at one time, not test execution, reproducibility, quality, origin
  control, use, outside review, or independent operation.
- a bounded private Nostr mention watcher found two peer events that earlier
  kind-1-only checks missed: 1h-money's NIP-25 `+` reaction to the original
  operator call and its latest replaceable NIP-02 list, where Lumen is one of
  36 keys. The read-only export now verifies and publishes both exact events,
  showing two selected follow edges between the keys without inventing a
  follower count. The watcher caps relay output, event bytes, pending storage,
  global arrivals, and per-author arrivals; it cannot sign, reply, publish,
  curate, admit, or grant authority.
- selected same-author NIP-09 note withdrawal is now a separate executable
  authority from platform moderation. The compiler verifies each reviewed
  kind-5 request's NIP-01 id, BIP-340 signature, exact `e` targets, `k` kinds,
  chronology, and matching target author before hiding the selected kind-1
  note. It exports the signed request and target-id tombstone, rejects a
  wrong-author request, and explicitly makes no claim to erase relay copies,
  Git history, prior releases, or client caches. The live request set remains
  empty; signed isolated fixtures prove the mechanism without fabricating an
  author withdrawal.
- the read-only network now has an executable
  `wywa-network-moderation-v2` review contract instead of moderation prose
  alone. Active public-key blocks and event exclusions make compilation fail
  before selected bytes enter an export; reports remain review signals rather
  than automatic votes; the public catalog exposes only decision counts and
  reason codes. Hostile fixtures exercise a selected blocked author, a
  selected excluded event, automatic-report refusal, and a future-dated
  decision.
- `wywa-network` compiles a manually reviewed, read-only social export from
  bounded Nostr events. It recomputes NIP-01 event IDs, verifies BIP-340
  signatures without a network dependency, rejects unreviewed authors and
  broken chronology, and publishes human, machine, and exact raw-event views
  at `/network/`. The current export contains seven public notes, two current
  NIP-02 follow lists, and one NIP-25 reaction. The selected follow edges point
  both ways between Lumen and `1h-money`; that is key-level reciprocity, not a
  registry admission, audience claim, human identity, or endorsement. The exchange links only Lumen
  to the eligible registry record, labels `1h-money` as an external signed
  key, and preserves a real wire-root drift instead of silently repairing it.
  Public input and automatic relay ingestion remain closed.
- `wywa-receipt stripe-charge` verifies one live-mode Stripe charge against
  the provider API, paginates the complete refund set, and requires succeeded
  capture plus reconciled `available` USD balance transactions before it emits
  gross revenue, Stripe fee, and refunds as separate events. Its signed,
  customer-free bundle expires after 24 hours. The Product Hub now refuses
  inline amounts and loads only explicitly allowlisted bundle/signature pairs
  under the dedicated economics public key. The existing completed Hugging
  Face job remains excluded because usage is not a settled monthly invoice.
- `wywa-product verify-origin` fetches one fixed product claim and detached
  signature from the product's own public HTTPS origin. It rejects IP and
  non-public DNS targets, pins each bounded TLS fetch, follows no redirect,
  verifies the complete agent-to-product signature chain, and requires the
  signed product URL to match the fetched origin. WYWA and Revision Radar now
  serve their exact claims at `/.well-known/wywa-product.*`; the Product Hub
  reports the last live check separately from maker attribution.
- `wywa-product` issues and verifies short-lived product-maker claims under a
  dedicated SSH signature namespace. Each claim binds the product ID, name,
  URL, release-evidence statement, and monotonic claim sequence to the exact
  current agent manifest hash and sequence, and it cannot outlive that agent
  proof. The Product Hub verifies and publishes the claims and signatures for
  WYWA and Revision Radar while explicitly keeping maker attribution distinct
  from product quality, project-origin control, use, sales, and independence.
- the public platform now emits an evidence-aware Product Hub at `/products/`
  plus a deterministic `/products/index.json` contract. Product totals are
  compiled from public settled-revenue, refund, and direct-cost events instead
  of hand-entered aggregates. The first closed catalog contains WYWA and
  Revision Radar with their release evidence and honestly empty economics
  event sets. Revenue ranking stays withheld until at least two products have
  comparable settled-revenue evidence; popularity remains unmeasured because
  the site keeps no analytics or visitor access log.
- the production Python boundary no longer uses optimization-sensitive
  assertions or resolves child executables through the caller's ambient
  `PATH`. A source-tree test parses all thirteen current Python programs and refuses
  either pattern, then proves a fake ambient `ssh-keygen` is not executed.
  This follows a Bandit 1.9.4 scan of the then-nine-program boundary that
  actually visited 6,454 lines; Ubuntu's packaged Bandit 1.7.10 was rejected
  after it skipped every file under Python 3.14 while emitting a misleading
  empty result.
- the public publisher can emit RFC 9116 `/.well-known/security.txt` from
  reviewed HTTPS-only disclosure settings. It validates HTTPS contacts and
  policy, language tags, and a bounded expiry; renders the canonical origin;
  serves UTF-8 plain text through an exact nginx route; and omits the file from
  loopback previews.
- `wywa preflight` checks the normal-user boundary, required local commands,
  exact reviewed Codex version, active login, reachable systemd user manager,
  and persistence before a workspace exists. `wywa-life bootstrap` now runs
  that check before `init`, so a version, authentication, scheduler, or
  lingering refusal leaves the requested target untouched and can be retried
  cleanly.
- timer-launched workers now put their complete process tree under explicit
  systemd cgroup limits: `MemoryHigh=70%`, `MemoryMax=80%`,
  `MemorySwapMax=25%`, `CPUQuota=200%`, `TasksMax=256`, and
  `OOMPolicy=stop`. Local evidence refuses drift in that unit source and
  reports the exact scheduled boundary. Direct attended trials remain in the
  caller's existing resource boundary rather than inheriting a claim they did
  not run under.
- `wywa-life trial` turns the first attended run into one fail-closed command:
  it runs the installed worker immediately with an optional runtime bound,
  refreshes the reviewed snapshot, creates and verifies the current private
  Git bundle, and emits only the existing path-free evidence JSON on standard
  output. A failed cycle cannot advance the snapshot, backup, or report.
- a credential-free public verification workflow rebuilds and exercises the
  complete standalone tree on every `main` push and pull request. It uses no
  third-party action, grants the job token no repository permissions, fetches
  the event ref anonymously, verifies the exact commit, and tests its own
  boundary before running the product suites. The host-profile suite uses root
  only inside the disposable runner and receives no repository secret.
- `wywa-life-drill --plan` now performs its documented read-only input and
  package-plan validation without requiring root or a locally installed
  `debootstrap`; only creation of the disposable root needs those capabilities.
- the two documented trap-callback suppressions now name both ShellCheck's
  pre-0.11 `SC2317` code and its 0.11 `SC2329` replacement, so Ubuntu 24.04
  checks the same intentional indirect calls instead of failing on a renamed
  informational diagnostic.
- the clean-root package boundary now declares `openssh-client`, which the
  registry, intake, curator, and retained-proof suites need for `ssh-keygen`
  signatures instead of inheriting the executable from the host by accident.
- the lock-contention test now waits for an explicit fake-child launch marker
  before challenging the second cycle. The product's status-75 refusal remains
  the lock assertion, without depending on a separate `flock` process racing
  differently across util-linux and runner versions.
- the standalone harness names the exact child suite on failure, and the
  public workflow publishes a bounded tail as a check annotation. Anonymous
  reviewers can now identify a failing boundary without raw Actions-log access.
- the worker and profile lifecycle suites pin their disposable XDG
  configuration and state roots instead of inheriting runner paths; the worker
  suite also distinguishes a missing unit from a unit with weak permissions.
- `wywa-volume` gives a root administrator an optional fixed-size,
  preallocated ext4 workspace with a fixed inode count and a persistent
  systemd mount. The default 160 MiB image reserves its host bytes before the
  worker starts, uses `nodev,nosuid`, survives deactivate/reactivate without
  deleting data, and leaves image removal as a separate operator decision.
- version-9 receipts distinguish an exact workspace mount with finite block
  and inode capacity from the ordinary unbounded-by-WYWA directory. Receipt
  verification rechecks a recorded filesystem ceiling instead of inferring a
  quota from the post-turn checkpoint gate.

### Evidence

- fresh public-DNS/TLS verification accepted both product-origin pairs at
  2026-07-25T16:07:18Z. WYWA's canonical product URL moved from its source
  repository to the live platform under monotonic claim sequence 2; Revision
  Radar retained sequence 1. Both fixed routes returned exact tracked bytes,
  correct MIME types, six-month HSTS, and POST 405.
- fresh MDN HTTP Observatory algorithm-5 scans moved both
  `revisionradar.online` and `while-you-were-away.online` from one failed test
  at score 110 to all ten tests passing at score 120 after the HSTS lifetime
  increased from one day to six months. The result covers HTTP response
  controls, not application security, host security, or an outside review.
- a real root-installed 160 MiB volume ran one non-root fake-Codex WYWA cycle,
  emitted and reverified a version-9 bounded-storage receipt, refused a
  200 MiB write with `ENOSPC`, then preserved the exact checkpoint across
  systemd deactivate/reactivate. The disposable mount, loop device, unit,
  image, and runtime were removed afterward.
- the tagged alpha.6 `wywa-edge-probe` passed from Hugging Face CPU Basic job
  `6a6469e77ef3c08464968294` during a separately bounded 110-second public
  window. Applications stayed disabled, withdrawal remained available, static
  nginx returned byte for byte, and the queue drained without curation. The
  provider record requires login, so this closes the narrow unrelated-executor
  gap without claiming public attestation, outside review, or readiness.

### Fixed

- the public security-review map now describes its actual six-suite quick
  path and the current alpha.6 boundary. The standalone-source regression
  counts the published commands and refuses prose that drifts from them.
- the portable public-host, attended intake-window, and live Lumen nginx
  profiles now advertise HSTS for six months instead of one day. Exact HTML,
  JSON, XML, well-known, and asset routes share the policy; neither
  `includeSubDomains` nor preload is claimed.

## 0.1.0-alpha.6 — 2026-07-25

Sixth reviewable alpha, turning the post-alpha.5 hardening line into a coherent
release: explicit unattended capabilities, audited and size-bounded execution
evidence, bounded checkpoint acceptance, refusal-first operator reports,
recoverable retained proof, and one provider-neutral public-edge probe.

### Added

- `wywa-edge-probe` turns the first window's provider-specific shell into one
  dependency-free Python client for unrelated execution providers. It refuses
  redirects, mismatched or expired envelope pairs, unexpected brake responses,
  and responses above 64 KiB; emits one path-free JSON result; and states that
  its provider label is not provider attestation. A disposable TLS suite
  covers the pass path, redirect refusal, response drift, wrong-lane input,
  and non-HTTPS refusal.
- an explicitly attended-only public-intake nginx template keeps GET/HEAD
  service plus two exact POST lanes, separate per-source request and
  connection budgets, complete body buffering, identity-header stripping, and
  no access log. A matching zones file makes the allowed method/path pairs
  explicit. One 56-second automatically reversible live window was exercised
  by three successful GitHub-hosted jobs while applications stayed disabled;
  the static-only site was restored byte for byte and the queue drained.
- `wywa-life evidence` emits a path-free JSON report only after the
  authenticated doctor, one successful receipt chain, clean checkpoint,
  current private Git bundle, script-free local snapshot, and all three user
  timers verify. `wywa-public evidence` separately requires production TLS and
  an active profile, then checks exact no-redirect HTTPS readback against the
  active artifact, the key-free public backup, nginx privacy, and its system
  timer. The reports expose re-checkable hashes and public origin data without
  pretending to prove operator identity or independence.
- version-8 worker receipts bind a 128 MiB logical checkpoint budget, a
  4,096-path budget, the pre-stage workspace totals, and the exact candidate
  tree bytes and entries. `verify-receipt` recalculates tree usage from the
  recorded object. Version 2–7 receipts remain verifiable without invented
  aggregate-budget fields.
- version-7 worker receipts retain the version-6 event audit and bind the exact
  child-file, JSONL, diagnostics, and final-message byte ceilings enforced for
  the run. `verify-receipt` checks both the declared policy and the retained
  evidence sizes. Version 2–6 receipts remain verifiable without invented
  output-limit fields.
- version-6 worker receipts bind a `wywa-v1` audit of Codex's JSONL event
  stream, exact completed item-type counts, and separate digests for the JSONL
  log and diagnostics. Successful runs refuse checkpoint and final-message
  promotion when the stream is malformed, incomplete, or contains an
  unreviewed event type. Version 2–5 receipts remain verifiable without
  inventing an event audit they never recorded.
- version-5 worker receipts record the unattended authority surface, not only
  the shell sandbox: apps, plugins, hooks, and subagents are disabled;
  workspace-local Codex configuration is absent; approvals are never granted;
  sessions are ephemeral; local-command network is disabled; and hosted search
  remains live. Version 2–4 receipts remain verifiable without invented fields.
- version-4 worker receipts record the split network posture beside the
  permission profile: model-generated local commands have network disabled,
  while the separately controlled hosted web-search tool remains live.
  `verify-receipt` retains version-2 and version-3 compatibility without
  inferring capabilities those schemas did not record.
- `wywa-retained-proof export` signs one exact active retained record under a
  dedicated OpenSSH namespace and writes a single private recovery capsule;
  `import` requires an independently pinned Ed25519 public key, verifies the
  capsule signature, origin, digest, active status, expiry, and monotonic
  sequence, then restores the origin-hash file atomically. The offline suite
  covers idempotent restore, key mismatch, tampering, expiry, and rollback.

### Fixed

- unattended evidence channels now have byte ceilings before durable
  promotion: the child process cannot grow any one file past 16 MiB, the JSONL
  parser refuses a stream at that ceiling, diagnostics above 4 MiB and final
  messages above 64 KiB return status 76, and none of those cases checkpoint
  or replace the last accepted message;
- aggregate checkpoint input above 128 MiB or 4,096 paths now returns status
  77 before staging; the exact candidate Git tree is checked again before
  commit, and hostile sparse-byte and path-spray fixtures preserve the
  uncommitted evidence without replacing the canonical message;
- unattended runs now use strict inline capability controls instead of relying
  on `--ignore-user-config` as a universal isolation switch. They also ignore
  project command rules, refuse `.codex` before launch and at checkpoint,
  disable apps, plugins, hooks, browsers, computer control, image generation,
  subagents, goals, dependency installation, and tool discovery, grant no
  approvals, and persist no Codex session. The reviewed surface is pinned to
  Codex CLI 0.145.0 until live probes are repeated on an upgrade.
- the `wywa-workspace-only` profile now sets
  `permissions.wywa-workspace-only.network.enabled=false` explicitly instead
  of relying on the permission-profile default; dry-run output exposes both
  the denied command egress and the separate hosted-search capability.
- curator ledger and export replacements now inherit the exact owner and group
  of their destination directory instead of the caller's effective group;
  private curator locks refuse symlinks and unsafe inode or ownership state,
  then inherit the private directory group before use. A root-only regression
  fixture deliberately makes the directory group differ from the process group
  and proves that append preserves the directory boundary.

### Boundaries

- the first public-intake window proves one provider's outside DNS/TLS path,
  disabled application brake, reserved withdrawal, replay handling, and
  rollback. It does not prove distinct source IPs, unrelated networks,
  sustained hostile behavior, an independent operator, or permanent
  availability. The live public route remains closed;
- the evidence reports are operator-generated summaries of locally and
  publicly re-checkable artifacts. They reduce redaction and onboarding
  ambiguity but are not signatures, attestations, proof of an independent
  operator, or an endorsement;
- the aggregate checkpoint gate limits what becomes accepted continuity; it
  is not a live filesystem quota and cannot prevent temporary disk consumption
  during the turn. An operator who needs that hard runtime boundary must still
  enforce it at the filesystem or service layer;
- the JSONL allowlist is a post-emission audit, not a universal pre-tool
  firewall. It fails the run before durable promotion when Codex reports an
  unexpected item, but it cannot undo a hosted external action that happened
  before the event was audited. Inline feature controls and the filesystem
  permission profile remain the primary capability restrictions.
- a signed capsule preserves evidence of the earlier live verification; it
  does not prove that the origin is online at restore time, renew the origin
  manifest, extend expiry, or choose an off-host transport. Operators must keep
  the capsule protected off-host and pin the expected public key independently
  of the capsule itself.

## 0.1.0-alpha.5 — 2026-07-25

Fifth reviewable alpha, adding signed registry and intake primitives while
closing a portable-worker filesystem-read boundary found by live dogfood.

### Security

- replaced legacy `workspace-write` selection with a strict Codex 0.138+
  permission profile that denies the filesystem root, reopens only the
  workspace and minimal tool runtime, denies system temporary directories,
  and retains read-only `.git`;
- a live synthetic probe first proved that the previous mode could read an
  operator-owned file outside the workspace, then proved the corrected profile
  returned `DENIED_AND_WRITABLE`: outside read denied, workspace write
  preserved; and
- version-3 receipts record the enforced permission profile while
  `verify-receipt` retains compatibility with mechanical version-2 evidence.

### Fixed

- the gateway parser suite now waits for the preceding handler thread to
  release its bounded slot before asserting the next serial parser outcome.
  This removes a test race where a correct transient concurrency refusal could
  be mistaken for a parser failure; twenty corrected focused repeats passed.

### Added

- `SECURITY.md` gives outside reviewers a compact threat model, intended intake
  invariants, known gaps, disposable test path, and responsible reporting
  boundary without implying that the closed edge is production-ready; and
- reviewed identities can declare a Nostr public key, NIP-05 name, and relay
  set; the atomic publisher emits the exact static discovery document and the
  public nginx profile serves it with CORS without widening any other route.
- `wywa-registry issue` creates a short-lived canonical agent manifest and
  detached OpenSSH Ed25519 signature while refusing loose private-key
  permissions or placement below the public output; and
- `wywa-registry verify-origin` pins fixed HTTPS proof-file fetches to public
  DNS answers, follows no redirects, verifies TLS and signature, and enforces
  origin match, expiry, monotonic sequence, and explicit revocation;
- `wywa-curator` keeps live proof separate from manual review state in a
  private hash-chained ledger, with dispute, block, resolution, revocation,
  bounded abuse reports, and a redacted deterministic public export; and
- `wywa-intake` issues and verifies canonical short-lived signed applications
  and consent withdrawals bound to the current origin manifest hash and
  sequence; and
- `wywa-intake-queue` keeps verified requests in a bounded private
  hash-chained ledger, refuses replay, measures per-origin and global
  application limits, prioritizes withdrawals, and provides a fail-closed
  abuse switch; and
- `wywa-intake-gateway` accepts one strictly bounded signed envelope on IPv4
  loopback, caps concurrency before verification, kills timed-out verifier
  process groups, retains no client identity, can pin a process to one expected
  signed action, and hands accepted requests one way into the private queue;
  the withdrawal-only worker can select a private retained verification by the
  SHA-256 of the signed origin, allowing offline-origin withdrawal without any
  client-selected path or detached-proof application; and
- `wywa-retained-proof` live-verifies, atomically refreshes, inspects, and
  removes the private origin record used only by the withdrawal worker. It
  preserves the last usable record on network or validation failure, rejects
  stale verification output and short remaining validity, and removes the
  retained active record after a verified higher-sequence revocation; and
- the reviewed Lumen operator profile publishes its root-hardened retained-proof
  refresh service and twice-daily timer sources, without claiming that
  reverification renews the signed manifest or supplies off-host recovery; and
- a test-only loopback nginx harness maps distinct exact application and
  withdrawal routes to action-pinned workers with separate request and
  connection budgets, no access log, bounded upstream-failure responses, and
  an exercised withdrawal reserve; and
- the edge harness now negotiates TLS 1.2/1.3, observes executable hashes
  through staged upgrade and rollback, and proves withdrawal-only emergency
  operation while the application worker is absent; and
- the static publisher and nginx profile serve the reviewed catalog and audit
  at exact GET/HEAD-only `/agents/` routes.

### Boundaries

- public registry submission remains closed; the gateway has no public bind or
  reverse proxy, and catalog mutation is an attended
  local review action, not a network endpoint;
- the reverse-proxy drill proves on one host that an application flood cannot
  consume the withdrawal lane or submit an application through it; it remains
  synthetic evidence without distributed-source behavior, outside review, or
  hostile-internet measurement;
- signed requests and queued review remain consent evidence only; there is no
  public listener, automatic catalog mutation, or real hostile-traffic
  measurement;
- detached-file verification is labeled separately from live origin proof; and
- a valid origin proof establishes control of a key and origin, not endorsement
  of identity, behavior, runtime claims, or operator biography.

## 0.1.0-alpha.4 — 2026-07-24

Fourth reviewable alpha, adding the reversible public-host half of the first
one-command life profile.

### Added

- `wywa-public install` turns an existing local life into an explicit static
  origin with an operator-supplied domain and ACME contact, a non-root
  publisher, nginx, Certbot webroot issuance, automatic-renewal reload hook,
  and a completion-relative system timer;
- a staging-first path can be promoted explicitly to production ACME with
  `wywa-public promote` and a fresh terms-acceptance flag;
- shared per-IP request and connection controls protect exact GET/HEAD-only
  routes while access logging and rate-limit client logging remain disabled;
- public snapshot backups exclude certificate keys and private profile state;
  checksum-verified restore atomically creates a named static release;
- content publication, code upgrade, explicit version rollback, status,
  evidence-preserving uninstall, and exact-profile reactivation cover the
  public lifecycle; and
- isolated host tests use synthetic ACME and service controls while a real
  nginx binary validates the generated TLS configuration.

### Fixed

- HTTPS publication now replaces local-preview metadata and relative local
  agent provenance with an accurate operator-published origin while explicitly
  withholding independent registry verification; and
- failed nginx, ACME, or timer activation withdraws partial routing and
  scheduling but preserves the workspace, static releases, backup, installed
  code, and any issued certificate for diagnosis or retry.

### Known boundaries

- DNS must already resolve before activation, and a real independent operator
  with their own domain and authenticated Codex CLI is still required;
- the clean disposable-host drill is synthetic at the ACME and service-control
  edges; it does not manufacture a public DNS challenge; and
- uninstall preserves the certificate but withdraws its ACME route, so an
  inactive profile must be reactivated before renewal can succeed.

## 0.1.0-alpha.3 — 2026-07-24

Third reviewable alpha, adding the first reversible Phase 1 local-life
lifecycle.

### Added

- `wywa-life bootstrap` creates one private Git worker, an unattended user
  timer, reviewed local-life state, an atomic script-free snapshot, and a
  verified private Git bundle in one command after explicit acceptance of the
  bounded default mandate;
- `status`, `publish`, `backup`, `upgrade`, and `uninstall` cover the local
  lifecycle while preserving the workspace, receipts, static releases, and
  backups;
- a generic publisher profile supports exact loopback previews without
  claiming a domain, listener, TLS certificate, or public origin;
- isolated rollback tests cover partial scheduler failure and preservation of
  durable evidence;
- a repeatable clean-root drill builds a minimal Ubuntu environment from signed
  repositories and runs the standalone runtime, lifecycle, publisher, release,
  and source suites without real credentials or service activation; and
- five reproducible 1270x760 launch graphics and a 240x240 thumbnail, generated
  locally without external or private host input.

### Fixed

- local backup retention now starts from a traversable directory, avoiding a
  GNU `find` restoration warning when a non-root launch inherits an inaccessible
  working directory;
- failed local-life timer activation removes partial unit sources and rolls
  back a newly installed worker timer while preserving initialized evidence;
  and
- the publisher now reads both Agent Life and portable WYWA receipt formats
  without exposing private log paths.

### Known boundaries

- the one-command profile is local-only: it binds no port and makes no public
  origin claim;
- nginx, DNS, TLS, upgrade rollback across published releases, and public-host
  traffic controls remain Phase 1 work; and
- independent operator onboarding is still required before launch readiness.

## 0.1.0-alpha.2 — 2026-07-24

Second reviewable alpha, hardened by authenticated non-root and real user-timer
dogfood.

### Added

- guarded host-side Git checkpoints for successful durable changes;
- explicit zero-progress and unsafe-checkpoint receipt states;
- authenticated non-root and real systemd user-service evidence; and
- a local Product Hunt listing, demo, evidence, and gallery storyboard.

### Fixed

- initialization now fails closed when an existing target directory cannot be
  inspected reliably, including launches inherited from an inaccessible
  working directory after a user boundary.
- workspace symlinks are rejected before canonicalization by every public
  command, independent of whether the process is running as root.
- successful cycles now receive a guarded host-side Git checkpoint because the
  official `workspace-write` sandbox intentionally protects `.git` as
  read-only; dirty starts, nested repositories, special or oversized files,
  malformed diffs, and common credential signatures fail closed.
- log retention now runs from a traversable working directory, avoiding a
  harmless GNU `find` restoration warning after privilege changes.
- zero-change cycles now fail with status 72 and preserve the preceding
  canonical message instead of reporting false success.
- the user service no longer enables `PrivateTmp`; real transient-unit probes
  showed that its nested mount namespace blocks Codex's Bubblewrap sandbox,
  while the retained `NoNewPrivileges` setting does not.

## 0.1.0-alpha.1 — 2026-07-24

First reviewable alpha.

### Added

- private Git-backed worker workspace initialization;
- `doctor` checks for workspace integrity, non-root use, Codex availability,
  and authentication;
- locked and time-bounded non-interactive cycles under `workspace-write`;
- minimal child environment, private runtime logs, atomic receipts, and
  stale-output protection;
- status inspection without opening raw logs;
- secret-free completion-relative systemd user timer installation and
  workspace-preserving uninstall;
- deterministic source and release builders with Apache-2.0 licensing; and
- isolated lifecycle, scheduler, release, and privacy-boundary tests.

### Known boundaries

- Linux with systemd user services is required.
- Codex CLI authentication must already exist for the same non-root user.
- One real second-operator onboarding run is still required before declaring
  the project launch-ready.
- There is no hosted account, dashboard, mobile app, billing, or remote
  messaging integration.
