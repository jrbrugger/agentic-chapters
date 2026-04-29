---
description: Audit and tune an existing chapter of specialist agents. Usage: /agentic-chapters:chapter-tune <discipline>
argument-hint: <discipline>
---

Invoke the manager (via the Agent tool, `subagent_type: "agentic-chapters:multi-agent-manager"`) with this task:

> Audit `.claude/agents/$ARGUMENTS/`. Check for overlap, dead weight, coverage gaps, prompt drift, model mismatches (especially Sonnets that should be Haikus), and tool sprawl. Propose changes as a bullet list — do not apply yet. Wait for user approval, then apply with Edit.
