---
description: Run a named playbook within a chapter end-to-end. Usage: /agentic-chapters:run-playbook <discipline> <playbook-name> <input> [--dry-run]
argument-hint: <discipline> <playbook-name> <input> [--dry-run]
---

Invoke the orchestrator (via the Agent tool, `subagent_type: "agentic-chapters:agent-orchestrator"`) **in playbook mode** with this task:

> Run the playbook named in $ARGUMENTS end-to-end.
>
> Parse $ARGUMENTS into: `<discipline>` (first token), `<playbook-name>` (second token), `<input>` (the rest, minus any flags), and flags (`--dry-run`).
>
> Read `.claude/agents/<discipline>/playbooks/<playbook-name>.md`. If missing, tell the user and suggest the architect mint it via `/agentic-chapters:new-chapter <discipline> "with a <playbook-name> playbook"`.
>
> Follow the playbook-mode procedure in your system prompt: initialize the run (generate `<run-id>`, create the run directory, write initial `state.json`), then for each phase dispatch specialists (parallel or sequential per the phase's body), wait for per-specialist artifacts, check for `[BLOCKING]` issues, run loops within the cap, verify exit criteria, surface gates, and advance.
>
> If `--dry-run` is set, print the plan only — do not invoke specialists, do not create the run directory.
>
> After the final phase, synthesize a single coherent reply. Reference the `<run-id>` so the user can inspect `state.json` and per-phase artifacts at `.claude/playbook-runs/<run-id>/`.
