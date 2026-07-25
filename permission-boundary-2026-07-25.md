# Portable filesystem permission boundary

Date: 2026-07-25 UTC

## Finding

WYWA previously selected Codex's legacy `workspace-write` sandbox and described
the portable worker as bounded to its workspace. That was too broad: the mode
restricted writes and network access, but a live synthetic probe on Codex CLI
0.145.0 successfully opened a root-owned mode-0600 file outside the workspace.
The file was empty and contained no credential. The result was `READABLE`.

This mattered beyond wording. An operator key stored elsewhere under the same
Unix identity could be read by model-generated commands even though the worker
could not write there. Environment scrubbing and checkpoint credential scans
were useful downstream controls, not a filesystem-read boundary.

## Corrected profile

The runtime now uses one strict inline permission profile:

- extend Codex's built-in `:workspace` profile so the workspace stays writable
  and `.git` retains its built-in protection;
- deny `:root` by default;
- reopen only `:minimal`, the runtime paths common tools need;
- deny `:tmpdir` and `:slash_tmp`; and
- keep sandboxed command network disabled.

The wrapper ignores unattended user configuration, passes the complete profile
on the command line, enables strict configuration parsing, and requires Codex
CLI 0.138.0 or newer. It does not combine the profile with the legacy
`--sandbox` option.

Permission profiles govern local command execution. Built-in web search,
connectors, browser tools, cloud tasks, and approved escalations are separate
capabilities; WYWA does not pretend the filesystem profile governs them.

## Verification

Two controlled probes used a disposable Git workspace and an empty synthetic
file outside it:

1. Legacy `workspace-write` opened the outside file: `READABLE`.
2. The exact replacement profile refused the outside open while allowing a new
   file in the workspace: `DENIED_AND_WRITABLE`.

The corrected probe was then repeated through `codex exec`, not only the
lower-level sandbox command. The model's attempted Python open failed, its
workspace patch succeeded, and the final result was
`DENIED_AND_WRITABLE`. A second empty sentinel placed beside the active
Codex authentication file was also refused through the exact production
arguments: `AUTH_PATH_DENIED`. No authentication file was opened, copied, or
printed. All disposable paths were removed.

`tests/test-wywa` now rejects a Codex version below 0.138.0, asserts every
profile argument, refuses any simultaneous legacy `--sandbox` flag, and checks
that version-3 receipts record `wywa-workspace-only`. The live probe remains an
upgrade gate because beta permission behavior cannot be proved by an argument
fixture alone.

## Evidence sources

Official OpenAI documentation, accessed 2026-07-25 UTC:

- Permission profiles, filesystem `read` / `write` / `deny` precedence,
  workspace-only example, scope, and Linux enforcement:
  https://learn.chatgpt.com/docs/permissions
- Codex configuration reference for `default_permissions`, named profiles, and
  the rule not to combine profiles with legacy sandbox settings:
  https://learn.chatgpt.com/docs/config-file/config-reference#configtoml

## Rollback

Revert the alpha.5 permission-boundary commit to restore the old behavior, but
do not describe that behavior as denying reads outside the workspace. A safer
operational rollback is to stop unattended timers until a corrected release is
installed.
