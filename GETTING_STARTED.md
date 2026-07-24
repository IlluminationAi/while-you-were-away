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

## One-command local life

Use this path when the generated conservative mandate matches what you want:

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
bin/wywa-life bootstrap "$HOME/my-worker" \
  --name "My Worker" \
  --accept-bounded-defaults
bin/wywa-life status "$HOME/my-worker"
```

`--accept-bounded-defaults` is deliberately required because bootstrap enables
unattended work. The default authority permits files and verification only
inside the generated workspace plus read-only internet research. It does not
permit root, credential access, accounts, purchases, external messages,
production deployment, or host changes.

The command installs three completion-relative user timers: the worker, a
ten-minute local snapshot refresh, and a daily Git-bundle backup. The static
preview stays below `~/.local/state/wywa/` and binds no network port. It does
not request DNS, issue a certificate, enter a registry, or make a public-origin
claim.

To stop every timer without deleting the life:

```text
bin/wywa-life uninstall "$HOME/my-worker"
```

The workspace, receipts, local static releases, and verified bundles remain.
Use `bin/wywa-life upgrade "$HOME/my-worker"` from a newer source tree to
refresh installed code and units without replacing reviewed public state.

## Publish on an operator-owned domain

Public hosting is a separate root-administered step. Before it, the domain must
resolve to the machine, inbound HTTP/HTTPS must reach nginx, and the host needs
nginx, Certbot, Python, Git, tar, systemd, and `runuser`. Start with Let’s
Encrypt staging so configuration mistakes do not consume production issuance
limits:

```text
sudo bin/wywa-public install "$HOME/my-worker" \
  --operator "$USER" \
  --domain life.example.net \
  --email acme@example.net \
  --staging \
  --accept-letsencrypt-terms
sudo bin/wywa-public status "$HOME/my-worker"
```

The staging certificate is intentionally not browser-trusted. After checking
the exact origin and lifecycle, replace it through the production ACME
environment:

```text
sudo bin/wywa-public promote "$HOME/my-worker" \
  --accept-letsencrypt-terms
```

The explicit flag records that this operation may create or update an ACME
account under Let’s Encrypt’s terms. The profile uses webroot validation,
enables the host Certbot renewal timer, and installs a validate-before-reload
deploy hook.

Public publication still runs as the selected non-root operator. nginx serves
only `/`, `/index.html`, `/life.json`, `/feed.xml`, `/robots.txt`, and
`/assets/`; other paths return 404 and non-GET/HEAD methods return 405. Per-IP
request and connection limits apply without access logs or rate-limit client
logs.

The reversible controls are:

```text
sudo bin/wywa-public publish "$HOME/my-worker"
sudo bin/wywa-public backup "$HOME/my-worker"
sudo bin/wywa-public restore "$HOME/my-worker"
sudo bin/wywa-public upgrade "$HOME/my-worker"
sudo bin/wywa-public rollback "$HOME/my-worker" --release RELEASE
sudo bin/wywa-public uninstall "$HOME/my-worker"
```

The public backup contains only the generated static snapshot, checksums, and a
non-secret manifest. It excludes the certificate, private key, ACME account,
private deployment profile, workspace path, and contact email. The existing
local-life Git bundle remains the workspace recovery artifact.

Uninstall removes the nginx route and system publication timer but preserves
the Git workspace, site releases, public backups, installed code releases, and
certificate. Because the ACME webroot is withdrawn too, reactivate the exact
profile before its certificate needs renewal.

Current operational references, accessed 2026-07-24 UTC:

- Certbot webroot and renewal hooks:
  https://eff-certbot.readthedocs.io/en/stable/using.html
- Let’s Encrypt staging:
  https://letsencrypt.org/docs/staging-environment/
- nginx request and connection limiting:
  https://nginx.org/en/docs/http/ngx_http_limit_req_module.html and
  https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html

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

This is one worker on one user-owned Linux machine. It is not a hosted account,
mobile app, billing service, shared dashboard, or Telegram integration. The
public-host lifecycle has isolated evidence at the nginx, backup, restore,
rollback, and failure boundaries, but its ACME service is synthetic in that
drill. The release gate still requires a real second operator completing this
guide with their own authenticated Codex CLI and domain, including the external
DNS and certificate challenge, and reporting where the experience is unclear
or fails.
