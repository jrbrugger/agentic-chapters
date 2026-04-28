---
description: Stand up a new chapter of specialist agents for a discipline. Usage: /new-chapter <discipline> [optional brief]
argument-hint: <discipline> [brief]
---

Invoke the `ai-agents-architect` agent (via the Agent tool, `subagent_type: "agentic-chapters:ai-agents-architect"`) with this task:

> Stand up the **$ARGUMENTS** chapter for this project.
>
> Follow the `chapter-bootstrap` skill. Survey the project (CLAUDE.md, AGENTS.md, package manifests, top-level structure, docs/), decide on specialists, write each agent file into `.claude/agents/<discipline>/` of the current working directory, write `CHAPTER.md`, and append the delegation reminder to the project's CLAUDE.md.
>
> Report back the roster, model distribution, and one-line example invocations.
