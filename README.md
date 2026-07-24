# While You Were Away

Working tagline: **Your AI keeps working after you close the tab.**

Status: `0.1.0-alpha.1`, ready for a real second-operator test but not yet
launch-ready.

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

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
bin/wywa init "$HOME/my-worker"
bin/wywa doctor "$HOME/my-worker"
bin/wywa run "$HOME/my-worker" --dry-run
```

Continue with `GETTING_STARTED.md` before enabling unattended work. Run the
self-contained checks with:

```text
tests/test-wywa
tests/test-release
tests/test-source-tree
```

Release changes and current limitations are in `CHANGELOG.md`.

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
- user Codex configuration is ignored during unattended runs, while existing
  Codex authentication is reused;
- only a small environment allowlist reaches the child process, so unrelated
  shell secrets do not leak into the worker;
- runtime state and logs live outside the agent-writable workspace;
- no API key, Telegram token, credential, or private Agent Life state is copied
  into a generated workspace;
- no service, timer, account, firewall rule, or public endpoint is installed by
  the current MVP.

The private root deployment remains a separate, explicitly authorized profile.
It is evidence for the product, not the default shipped security posture.

## First-release acceptance criteria

The first useful release is complete only when:

1. one command creates a clean, Git-backed worker workspace with no secret or
   private Agent Life data;
2. `doctor` proves the workspace, Git, Codex binary, Codex login, private state
   directory, and non-root boundary are ready;
3. `run` uses an exclusive lock, bounded runtime, signal forwarding, private
   logs, atomic last-message publication, and `workspace-write`;
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
(acceptance criteria 1–7) is implemented:

- `init` creates a private workspace, explicit authority files, and a clean
  initial Git checkpoint;
- `doctor` enforces required files, Git integrity, Codex login, safe runtime
  storage, and the non-root default;
- `run` uses `workspace-write`, ignores unattended user configuration, scrubs
  unrelated environment variables, bounds runtime, forwards signals, excludes
  concurrent cycles, and atomically publishes success-only final messages plus
  a result receipt; and
- `status` exposes the receipt without requiring raw-log access.

`install-user` installs the CLI and templates into the user's home and enables
a secret-free, completion-relative systemd user timer. The unit carries only a
workspace hash; the private state record resolves it to the path. It retains
`NoNewPrivileges` and a private temporary directory. Persistent installation
requires user lingering unless `--session-only` is explicit, so the product
does not silently stop after an SSH logout. `uninstall-user` disables and
removes only that workspace's units while preserving its Git workspace and
runtime receipts.

`tests/test-wywa` exercises unsafe initialization targets, root refusal,
authentication failure, runtime symlinks, success, partial failure, timeout,
stale-output preservation, secret filtering, lock contention, orphan cleanup,
timer generation, idempotent reinstall, scheduled dispatch, and rollback.
ShellCheck passes. The complete Agent Life self-check, including radar,
recovery, bootstrap, backup, Telegram, package, service, and live-site
boundaries, passes with zero failures and warnings.

Acceptance criterion 8 also has a real fresh-account proof. A disposable
non-root `wywaprobe` account initialized a worker, passed `doctor`, installed a
real systemd user timer, ran the service through the user manager with the fake
Codex fixture, returned status 0 under `workspace-write`, and passed user-unit
verification. An initial attempt exposed privileged capability-hardening
directives that user managers cannot apply; those directives were removed
while `NoNewPrivileges` and `PrivateTmp` remained. The successful repeat left
no credential or workspace path in the units. The timer, user manager, linger,
home, and account were removed after the proof.

`bin/build-release` now produces deterministic
`while-you-were-away-0.1.0-alpha.1.tar.gz` and SHA-256 artifacts from an
explicit allowlist. The release test rebuilds twice byte-for-byte, verifies the
checksum, extracts it, runs ShellCheck, initializes a clean workspace through
the packaged CLI, and rejects private deployment markers.

`bin/build-source-tree` now emits a deterministic standalone repository with
the CLI, templates, Apache-2.0 license, documentation, and self-contained tests.
Two independently built trees compare byte-for-byte, pass all three tests, and
reject private deployment markers before publication.

## Planned interface

```text
wywa init WORKSPACE
wywa doctor WORKSPACE
wywa run WORKSPACE [--max-runtime 55m] [--dry-run]
wywa status WORKSPACE
wywa install-user WORKSPACE [--dry-run]
wywa uninstall-user WORKSPACE
```

The CLI delegates model authentication to an already authenticated Codex CLI.
It does not call the OpenAI API directly and does not create, read, or print API
keys.

## License

Apache License 2.0. See `LICENSE`.

## Next action

Exercise `GETTING_STARTED.md` and the reproducible alpha artifact with a real
second operator: authenticated install, one scheduled cycle, status, and
uninstall. This is the one acceptance criterion that cannot be satisfied by
another synthetic local account. Use the resulting friction log to decide
whether the next surface should be a guided installer, a small local dashboard,
or both.

## Rollback

All product work is contained below this directory plus explicit test and index
entries. Code rollback is `git revert <product-commit>`. Generated workspaces
are independent Git repositories and are never deleted by `wywa`.
