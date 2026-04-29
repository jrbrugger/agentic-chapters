---
description: Run a named pipeline within a chapter end-to-end. Usage: /agentic-chapters:run-pipeline <discipline> <pipeline-name> <input> [--dry-run]
argument-hint: <discipline> <pipeline-name> <input> [--dry-run]
---

Invoke the orchestrator (via the Agent tool, `subagent_type: "agentic-chapters:agent-orchestrator"`) **in pipeline mode** with this task:

> Run the pipeline named in $ARGUMENTS end-to-end.
>
> Parse $ARGUMENTS into: `<discipline>` (first token), `<pipeline-name>` (second token), `<input>` (the rest, minus any flags), and flags (`--dry-run`).
>
> Read `.claude/agents/<discipline>/pipelines/<pipeline-name>.md`. If missing, tell the user and suggest the architect mint it via `/agentic-chapters:new-chapter <discipline> "with a <pipeline-name> pipeline"`.
>
> Follow the pipeline-mode procedure in your system prompt: initialize the run (generate `<run-id>`, create the run directory, write initial `state.json`), then for each phase dispatch specialists (parallel or sequential per the phase's body), wait for per-specialist artifacts, check for `[BLOCKING]` issues, run loops within the cap, verify exit criteria, surface gates, and advance.
>
> If `--dry-run` is set, print the plan only — do not invoke specialists, do not create the run directory.
>
> After the final phase, synthesize a single coherent reply. Reference the `<run-id>` so the user can inspect `state.json` and per-phase artifacts at `.claude/pipeline-runs/<run-id>/`.
