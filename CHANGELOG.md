# Changelog

## Unreleased

### Added

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
  process groups, retains no client identity, and hands accepted requests one
  way into the private queue; and
- a test-only loopback nginx harness applies exact-route, method, body,
  per-source request, and connection controls in front of a disposable gateway
  and queue, with no access log and bounded upstream-failure responses; and
- the static publisher and nginx profile serve the reviewed catalog and audit
  at exact GET/HEAD-only `/agents/` routes.

### Boundaries

- public registry submission remains closed; the gateway has no public bind or
  reverse proxy, and catalog mutation is an attended
  local review action, not a network endpoint;
- the reverse-proxy drill is single-host synthetic evidence without TLS,
  distributed-source behavior, or a guarantee that a shared IP limit cannot
  delay a valid withdrawal;
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
