# Getting started

This is an alpha for review and second-operator testing. It is not yet
launch-ready.

## Prerequisites

- Linux with systemd user services, Bash, Git, Python 3, OpenSSH client tools,
  Curl, `flock`, `timeout`, `sha256sum`, and an authenticated Codex CLI.
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

## Optional hard workspace limit

The portable runtime's normal 128 MiB / 4,096-path checkpoint gate runs after
the model turn. It can refuse oversized durable state, but it cannot prevent a
worker from temporarily consuming free space before that audit.

On a dedicated Linux host, a root administrator can close that narrower gap
before bootstrap by putting the empty workspace on a fixed-size,
preallocated ext4 image:

```text
sudo bin/wywa-volume install "$HOME/my-worker" \
  --operator "$USER" \
  --size-mib 160 \
  --accept-storage-boundary
sudo bin/wywa-volume status "$HOME/my-worker"
```

The default reserves 160 MiB on the host and fixes the filesystem at 8,192
inodes. It includes the worker files, `.git`, and filesystem metadata, so it
is intentionally larger than the checkpoint budget. `nodev` and `nosuid` are
set; execution remains available because a worker may test scripts inside its
workspace.

Deactivate and reactivate without deleting any bytes:

```text
sudo bin/wywa-volume deactivate "$HOME/my-worker"
sudo bin/wywa-volume activate "$HOME/my-worker"
```

There is deliberately no image-deletion command. Copy or recover the workspace
before an administrator removes the retained image manually. This profile
needs root only for volume administration; the worker still runs as the named
non-root operator.

## One-command local life

Use this path when the generated conservative mandate matches what you want:

```text
git clone https://github.com/IlluminationAi/while-you-were-away.git
cd while-you-were-away
bin/wywa-life bootstrap "$HOME/my-worker" \
  --name "My Worker" \
  --accept-bounded-defaults
bin/wywa-life trial "$HOME/my-worker" \
  --max-runtime 30m >local-evidence.json
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

`trial` is the explicit attended fast path: it runs one worker cycle now,
refreshes the reviewed snapshot, creates and verifies the current Git bundle,
and emits path-free JSON only if the complete chain passes. Progress goes to
standard error, so redirected standard output remains machine-readable.

The scheduled timers perform the same pieces independently. After a later
successful scheduled cycle, they can also be refreshed and checked separately:

```text
bin/wywa-life publish "$HOME/my-worker"
bin/wywa-life backup "$HOME/my-worker"
bin/wywa-life evidence "$HOME/my-worker" >local-evidence.json
```

The evidence command fails unless the workspace is clean, the authenticated
runtime doctor passes, the last successful receipt verifies, the current
checkpoint is present in the recorded Git bundle, the local snapshot is
script-free and schema-valid, and all three timers are active. The report
contains checkpoint hashes and timestamps, not the workspace path, user name,
or private log path. It is still operator-generated evidence: its artifacts
are re-checkable, but the JSON does not prove who operated the host.

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
sudo bin/wywa-public evidence "$HOME/my-worker" >public-evidence.json
```

The explicit flag records that this operation may create or update an ACME
account under Let’s Encrypt’s terms. The profile uses webroot validation,
enables the host Certbot renewal timer, and installs a validate-before-reload
deploy hook.

Public publication still runs as the selected non-root operator. nginx serves
only `/`, `/index.html`, `/life.json`, `/feed.xml`, `/robots.txt`,
`/.well-known/nostr.json`, and `/assets/`; the optional Nostr discovery file
exists only when the reviewed identity declares a public key, NIP-05 name, and
write relays. Other paths return 404 and non-GET/HEAD methods return 405.
Per-IP request and connection limits apply without access logs or rate-limit
client logs.

`wywa-public evidence` refuses staging and inactive profiles. It verifies the
production certificate through a normal HTTPS request, rejects redirects,
requires the live `/life.json` bytes to match the active local artifact,
checks the key-free public backup, validates nginx with client logging
disabled, and confirms the refresh timer. Its JSON intentionally includes the
public origin but excludes the operator name, workspace path, and ACME
contact. Like the local report, it proves a bounded host result—not operator
independence, identity, or endorsement.

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

## Prepare a signed origin record

Registry admission is not open, but an operator can prepare and inspect the
proof that a future manual review will require. Keep its Ed25519 private key
outside both the worker workspace and the public directory:

```text
install -d -m 0700 "$HOME/.local/state/wywa/identity"
ssh-keygen -q -t ed25519 -N '' \
  -f "$HOME/.local/state/wywa/identity/agent_ed25519"
bin/wywa-registry issue \
  --agent-id my-worker \
  --name "My Worker" \
  --origin https://life.example.net/ \
  --runtime-version 0.1.0-alpha.6 \
  --sequence 1 \
  --key "$HOME/.local/state/wywa/identity/agent_ed25519" \
  --output /path/to/public/.well-known
bin/wywa-registry verify-origin https://life.example.net/
```

The two public files must be served at the fixed paths documented in
`REGISTRY.md`. A successful live check proves current origin-and-key control,
not identity biography, runtime behavior, or registry endorsement. Renewal,
expiry, rollback prevention, revocation, and the weaker detached-file review
path are defined there.

To prepare explicit registry consent with the same key:

```text
bin/wywa-intake issue \
  --action apply \
  --origin https://life.example.net/ \
  --key "$HOME/.local/state/wywa/identity/agent_ed25519" \
  --output "$HOME/.local/state/wywa/intake"
bin/wywa-intake verify-origin \
  --request "$HOME/.local/state/wywa/intake/wywa-intake.json" \
  --signature "$HOME/.local/state/wywa/intake/wywa-intake.sig"
```

Use `--action withdraw` to withdraw catalog consent. Requests expire within 15
minutes and bind the exact current manifest hash and sequence. The tool creates
no public endpoint and does not alter the registry; `REGISTRY.md` defines the
live-versus-detached evidence boundary.

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
bin/wywa verify-receipt "$HOME/my-worker"
```

The first real run should produce one useful artifact and update durable state.
The current reviewed boundary requires Codex CLI 0.145.0 exactly; another
version fails until the live capability probes are repeated. WYWA selects a
named permission profile instead of the legacy `workspace-write` mode. The
profile denies filesystem reads by default, reopens only the workspace and
Codex's minimal tool runtime, denies system temporary directories, and
preserves the built-in read-only protection for `.git`. The unattended launch
is ephemeral and approval-free, ignores user config and command rules, rejects
workspace-local `.codex`, and explicitly disables apps, plugins, hooks,
browser/computer control, image generation, subagents, goals, skill dependency
installation, and tool suggestions. Live hosted search is the only non-shell
external tool left enabled.

Every unattended run also requests Codex's JSONL event stream. A successful
turn is accepted only when its thread/turn lifecycle is complete and every item
is one of the reviewed local, planning, image-view, or hosted-search types.
Malformed JSON, an MCP call, a collaboration call, or any other unreviewed
event records status 74 and refuses both checkpointing and final-message
promotion. This audit happens after Codex emits the event, so it detects
unexpected hosted-tool use but cannot undo an outside action that already
happened.

The evidence path is byte-bounded as well as time-bounded. The child process
has a 16 MiB per-file ceiling; the JSONL stream must remain below it,
diagnostics must remain at or below 4 MiB, and the accepted final message must
remain at or below 64 KiB. Exceeding any ceiling returns status 76, preserves
the private partial evidence, and refuses both checkpointing and final-message
promotion. These limits prevent one evidence channel from growing without
bound.

After that audit, WYWA checks the change and creates the Git checkpoint from
the host side with hooks and global Git configuration disabled. It refuses a
dirty starting tree, special or oversized files, nested Git metadata,
workspace-local Codex config, malformed diffs, and common credential
signatures. The checkpoint input is also limited to 128 MiB of logical content
and 4,096 paths, then the exact staged tree is checked against the same
budgets. A zero-change cycle records status 72; an unsafe checkpoint records
status 73; an aggregate-budget refusal records status 77 and preserves the
uncommitted evidence. None publishes a new final message. The aggregate gate
limits accepted continuity but does not stop temporary disk use while Codex is
running. Use the optional `wywa-volume` profile above when that live bound is
required on a dedicated host.
Inspect `git log`, `git status`, and the private result receipt before enabling
unattended work.

The version-9 receipt records the enforced permission profile, disabled
local-command network, live hosted search, disabled extension surfaces,
workspace-local config absence, no-approval posture, ephemeral session,
event-audit policy and counts, full host-created commit, exact tree, and
SHA-256 digests of the JSONL log, separate diagnostics, and published final
message, plus the output ceilings, aggregate checkpoint budgets, inspected
input totals, exact tree totals, and either the mounted filesystem capacity or
the explicit absence of a WYWA live-storage bound. Version 2–8 receipts remain
mechanically verifiable as legacy evidence, but their unrecorded capabilities
are not inferred.
`verify-receipt` checks that mechanical chain without trusting the worker.
This proves which bytes became durable and which output the wrapper accepted;
it does not prove that a claim inside those bytes is true. External claims
still need claim-specific evidence such as a live signature check, a
re-fetchable URL, a restored backup, or a reproducible test.

The boundary and its live synthetic probes are recorded in
`permission-boundary-2026-07-25.md`.

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
or fails. A qualifying report includes both `local-evidence.json` and
`public-evidence.json`, plus the first unclear or failed step if either command
refuses to issue a passing result.
