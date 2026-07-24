# Getting started

This is an alpha for review and second-operator testing. It is not yet
launch-ready.

## Prerequisites

- Linux with systemd user services, Bash, Git, `flock`, `timeout`,
  `sha256sum`, and an authenticated Codex CLI.
- A normal non-root user. The portable runtime refuses root by default.
- User lingering if the worker must survive an SSH logout. An administrator
  can inspect it with:

```text
loginctl show-user "$USER" -p Linger
```

If it reports `Linger=no`, ask the administrator to enable lingering for the
account. WYWA will otherwise refuse persistent installation. The explicit
`--session-only` option is available for a desktop session where work after
logout is not required.

WYWA reuses the Codex CLI's existing authentication. It does not ask for,
create, read, or store an API key. Confirm the CLI before continuing:

```text
codex --version
codex login status
```

Install and authentication documentation:

- https://developers.openai.com/codex/cli/
- https://developers.openai.com/codex/auth/

## Create one worker

Clone the source and enter the project directory:

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
```

Create the worker:

```text
bin/wywa init "$HOME/my-worker"
bin/wywa doctor "$HOME/my-worker"
```

Read and edit the generated `MANDATE.md`, `CONSTITUTION.md`, and
`state/CURRENT.md`. The defaults permit work only inside that workspace and do
not authorize deployment, messaging, purchases, credentials, root access, or
host changes.

Preview the execution boundary, then run one attended cycle:

```text
bin/wywa run "$HOME/my-worker" --dry-run
bin/wywa run "$HOME/my-worker"
bin/wywa status "$HOME/my-worker"
```

The first real run should produce one useful artifact and update durable state.
Codex cannot write `.git` inside `workspace-write`; after a successful cycle,
WYWA checks the change and creates the Git checkpoint from the host side with
hooks and global Git configuration disabled. It refuses a dirty starting tree,
special or oversized files, nested Git metadata, malformed diffs, and common
credential signatures. A zero-change cycle records status 72; an unsafe
checkpoint records status 73 and preserves the uncommitted evidence. Neither
case publishes a new final message. Inspect `git log`, `git status`, and the
private result receipt before enabling unattended work.

## Keep it working after the terminal closes

```text
bin/wywa install-user "$HOME/my-worker" --dry-run
bin/wywa install-user "$HOME/my-worker"
systemctl --user list-timers "wywa-*"
```

The timer first wakes after boot and then one hour after the preceding cycle
finishes. This completion-relative cadence prevents an overlong run from
causing an immediate catch-up loop.

The unit contains a workspace hash, not the workspace path or a credential.
Private path resolution, logs, the lock, the last message, and receipts live
under `~/.local/state/wywa/`. The worker itself is installed below
`~/.local/lib/wywa/`.

## Inspect and stop

```text
~/.local/lib/wywa/bin/wywa status "$HOME/my-worker"
~/.local/lib/wywa/bin/wywa uninstall-user "$HOME/my-worker"
```

Uninstall stops and removes only that workspace's service and timer. It
preserves the Git workspace and private runtime evidence. If an administrator
enabled lingering solely for this product and the account has no other user
services that need it, ask the administrator to disable lingering separately.

## Known boundary

This is one worker on one user-owned Linux machine. It has no hosted account,
mobile app, billing, shared dashboard, Telegram integration, or public launch
flow. The current release gate is a real second operator completing this guide
with their own authenticated Codex CLI and reporting where the experience is
unclear or fails.
