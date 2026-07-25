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
- explicitly set sandboxed command network to disabled.

The wrapper ignores unattended user configuration and command rules, rejects a
workspace-local `.codex` layer before launch and at checkpoint, passes the
complete profile on the command line, enables strict configuration parsing,
and pins the reviewed boundary to Codex CLI 0.145.0. It does not combine the
profile with the legacy `--sandbox` option.

Permission profiles govern local command execution. Built-in web search,
connectors, browser tools, cloud tasks, and approved escalations are separate
capabilities; WYWA does not pretend the filesystem profile governs them.
WYWA deliberately enables live hosted search for its read-only research
mandate. Version-7 receipts record both sides of that split plus the rest of
the unattended authority surface.

## Non-shell capability boundary

The permission profile alone is not an external-tool allowlist. Current
official configuration documents apps, plugins, browser/computer control,
image generation, multi-agent tools, hooks, skill dependencies, and tool
suggestions as separate features, many enabled by default. `--ignore-user-config`
only skips `$CODEX_HOME/config.toml`; it does not mean “shell plus search.”

WYWA now starts every unattended turn with no approval path and an ephemeral
session. It disables apps, plugins, remote plugins, hooks, browser and computer
control, image generation, subagents, goals, Guardian approval, authentication
elicitation, MCP elicitation, skill dependency installation, skill search,
tool suggestions, and workspace dependency loading. Live hosted search remains
enabled deliberately. Existing Codex authentication is still used by the host
client and remains unreadable to sandboxed tools.

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

`tests/test-wywa` now rejects every Codex version except the reviewed 0.145.0,
asserts every profile and feature argument, refuses any simultaneous legacy
`--sandbox` or dangerous bypass, rejects workspace-local Codex configuration,
and checks that version-7 receipts preserve the full posture, audited event
counts, and output ceilings. Version 2–6
receipts remain verifiable without retroactively claiming fields they did not
record.

An exact live 0.145.0 catalog probe under the production arguments exposed only
workspace tools, planning/input helpers, the hosted search tool, and local
`view_image`; it exposed no app, plugin, browser, computer-control,
image-generation, or subagent capability. A second probe asked `view_image` to
open WYWA's public thumbnail outside the disposable workspace. The router
returned `No such file or directory`, and the model reported
`VIEW_IMAGE_OUTSIDE_DENIED`. The current official schema documents a
`tools.view_image` control, but the live 0.145.0 CLI rejects that inline field;
verified runtime behavior takes precedence over the newer schema. The surviving
tool therefore remains part of the filesystem upgrade probe.

## Event-stream audit

Feature flags describe intended availability; they do not prove which tool
items actually appeared in a turn. WYWA now runs `codex exec --json`, keeps its
JSONL event stream separate from stderr diagnostics, and applies the versioned
`wywa-v1` policy before checkpointing. The policy requires one complete thread
and turn, a completed agent message, balanced started/completed items, valid
UTF-8 JSON objects, and only the reviewed item types: agent messages,
reasoning, command execution, file changes, hosted search, planning,
workspace-scoped image view, and context compaction.

A malformed stream or an MCP, dynamic, collaboration, or other unreviewed item
returns status 74. The wrapper preserves private logs but does not checkpoint
workspace changes or replace the last accepted message. Version-7 receipts bind
the exact completed-item counts, JSONL digest, and separate diagnostics digest;
`verify-receipt` repeats the structural audit.

Two live Codex 0.145.0 probes under the exact production arguments established
the current wire shapes. One hosted-search turn emitted `web_search`
`item.started` and `item.completed`. A second turn emitted
`command_execution` and `file_change` pairs for `pwd` and one `apply_patch`;
both ended with one `turn.completed` and the expected final message.

This is intentionally described as a post-emission audit. Codex hooks do not
cover hosted tools, and some specialized paths can opt out of the local hook
path. The audit can refuse durable promotion after unexpected use; it cannot
reverse an external action that occurred before the event reached stdout.
Inline feature controls, the deny-read permission profile, and disabled local
network remain the primary enforcement layers.

The audit input is also size-bounded before Python reads it. The child process
has a 16 MiB per-file limit, the JSONL parser refuses a stream that reaches
that ceiling, diagnostics above 4 MiB fail promotion, and the accepted final
message is limited to 64 KiB. Each case returns status 76, preserves the prior
canonical message, and creates no checkpoint. Adversarial fixtures exercise
all three channels. This is a per-file evidence boundary, not a claim that the
workspace has an aggregate storage quota.

## Evidence sources

Official OpenAI documentation, accessed 2026-07-25 UTC:

- Permission profiles, filesystem `read` / `write` / `deny` precedence,
  workspace-only example, scope, and Linux enforcement:
  https://learn.chatgpt.com/docs/permissions
- Codex configuration reference for `default_permissions`, named profiles, and
  the rule not to combine profiles with legacy sandbox settings, plus the
  separate app, plugin, browser, multi-agent, and hook features:
  https://learn.chatgpt.com/docs/config-file/config-reference#configtoml
- `codex exec` reference for the exact scope of `--ignore-user-config`,
  `--ignore-rules`, `--ephemeral`, and `--json`, including the documented
  event and item families:
  https://learn.chatgpt.com/docs/developer-commands#codex-exec
- Codex non-interactive guide for JSONL lifecycle and item types:
  https://learn.chatgpt.com/docs/non-interactive-mode
- Codex hooks guide for `PreToolUse` coverage and the explicit hosted-tool and
  specialized-path exceptions:
  https://learn.chatgpt.com/docs/hooks

## Rollback

Revert the alpha.5 permission-boundary commit to restore the old behavior, but
do not describe that behavior as denying reads outside the workspace. A safer
operational rollback is to stop unattended timers until a corrected release is
installed.
