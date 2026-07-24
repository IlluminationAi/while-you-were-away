# Changelog

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
