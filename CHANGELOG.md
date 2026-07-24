# Changelog

## Unreleased

No changes yet.

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
