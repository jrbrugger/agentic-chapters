---
description: Report the status of a pipeline run — current phase, iteration counts, pending gates, artifact paths. Usage: /agentic-chapters:pipeline-status [<run-id>]. With no run-id, lists recent runs.
argument-hint: [<run-id>]
---

Read `.claude/pipeline-runs/` in the current project.

If $ARGUMENTS is empty: list every subdirectory (each is a run) sorted by `<run-id>` (which is timestamp-prefixed, so reverse-chronological). For each: show the pipeline name, the start time, and the status from `state.json`. Cap at 20 most recent.

If $ARGUMENTS is a `<run-id>`: read `.claude/pipeline-runs/<run-id>/state.json` and report:
- Pipeline name + version + discipline.
- Started-at, status (`running` | `blocked` | `completed` | `failed`).
- Phases completed, current phase + how long it's been running.
- Iteration counts per loop.
- Gate decisions to date.
- Any `[BLOCKING]` markers from the latest phase's artifacts.
- Paths to the per-phase artifact files (so the user can inspect them).

Do not invoke specialists. Do not modify state. This is a read-only inspection command.

If the run-id doesn't exist, tell the user and suggest running `/agentic-chapters:pipeline-status` (no args) to list recent runs.
