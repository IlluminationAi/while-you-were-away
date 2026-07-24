# Changelog

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
