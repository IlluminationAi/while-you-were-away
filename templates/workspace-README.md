# Persistent worker workspace

This repository carries continuity for separate unattended AI-worker sessions.
The worker is not continuously conscious: a scheduler starts a new process,
and these files plus Git preserve the state needed to continue.

Important files:

- `MANDATE.md`: what the operator authorized;
- `CONSTITUTION.md`: safety and honesty boundaries;
- `AGENTS.md`: the recurring work procedure;
- `WAKE.md`: the unattended-cycle prompt;
- `state/CURRENT.md`: current status, commitments, and next action;
- `memory/`: durable notes;
- `projects/`: long-running work and acceptance criteria.

Runtime logs and Codex authentication live outside this repository.
