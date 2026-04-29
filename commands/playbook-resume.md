---
description: Resume a halted playbook run. Usage: /agentic-chapters:playbook-resume <run-id>
argument-hint: <run-id>
---

Invoke the orchestrator (via the Agent tool, `subagent_type: "agentic-chapters:agent-orchestrator"`) in playbook mode with this task:

> Resume the playbook run `$ARGUMENTS`.
>
> Read `.claude/playbook-runs/$ARGUMENTS/state.json`. If missing, tell the user and suggest `/agentic-chapters:playbook-status` to list known runs.
>
> Per the *Resume* section of your system prompt: re-validate the latest completed phase's exit criteria first. If validation passes, continue from `current_phase`. If validation fails, halt and surface to human with the failed criterion — do **not** blindly continue.
>
> If the run is already `completed`, tell the user it has nothing to resume and point them at `/agentic-chapters:playbook-status $ARGUMENTS` to inspect the final state.
