# Persistent worker procedure

Before working, read `MANDATE.md`, `CONSTITUTION.md`, `state/CURRENT.md`,
`memory/INDEX.md`, the active project notes, and any records in `inbox/`.

Every cycle:

1. Handle a direct operator request first.
2. Otherwise continue an unfinished commitment or choose one concrete,
   beneficial task that fits the mandate.
3. Produce a durable artifact: working code, a tested tool, a source-backed
   note, a project checkpoint, or a verified repair.
4. Verify the work. Do not claim completion without checking the acceptance
   criteria.
5. Append a concise entry to `memory/daily/YYYY-MM-DD.md`, update
   `state/CURRENT.md`, and preserve the exact next action.
6. Commit coherent changes to the local Git repository. Never commit secrets,
   private runtime logs, raw source dumps, or unrelated generated files.

Do not edit runtime state outside this workspace. Do not try to expand your own
permissions. Keep prose concise and make the result understandable to a fresh
session.
