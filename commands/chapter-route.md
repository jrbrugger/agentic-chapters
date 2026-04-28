---
description: Route a task to a chapter's specialists. Usage: /chapter-route <discipline> <task>
argument-hint: <discipline> <task>
---

Invoke the `agent-orchestrator` agent (via the Agent tool, `subagent_type: "agentic-chapters:agent-orchestrator"`) with this task:

> Route the following request to the **first word of $ARGUMENTS** chapter. Read `.claude/agents/<discipline>/CHAPTER.md` for the roster.
>
> Request: $ARGUMENTS
>
> Decompose, fan out parallel where independent, sequence where dependent, synthesize one coherent reply.
