---
name: chapter-bootstrap
description: Recipe the ai-agents-architect follows to stand up a new chapter. Lists the survey checklist, the per-specialist fields, the files to write, and the report shape. Invoke when designing any new agentic chapter.
---

# Chapter bootstrap recipe

When the architect creates a new chapter, follow these steps in order. Every chapter has the same shape so the orchestrator and manager can operate on any of them.

## 1. Survey the project (read in parallel)

In a single message with multiple Read calls:
- `CLAUDE.md`, `AGENTS.md`, `README.md`
- Package manifests: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`
- Top-level directory tree, one level deep (`Glob` `*` with no recursion)
- `docs/` table of contents if present

This survey determines the roster. Don't skip it — generic chapters that ignore the project produce generic agents that the orchestrator can't route effectively.

## 2. Identify work areas for the named discipline

For *this specific* project. Examples for engineering on a Next.js + Supabase project:
- Backend / API (Server Actions, Route Handlers)
- Frontend / UI (App Router, Server vs. Client components)
- Database (Supabase, RLS, migrations)
- Auth & security (Supabase Auth, multi-tenancy)
- Testing
- Performance / caching
- Payments / integrations (Stripe)
- Language specialists for the dominant languages in the repo (e.g. `engineering-typescript`, `engineering-python`, `engineering-rust`) — these handle idiom-level concerns: type-system tricks, async patterns, build/lint configs, language-specific perf gotchas. Add a language specialist when the repo has non-trivial code in that language.

For a Rust CLI, it would look completely different (e.g. `engineering-rust`, `engineering-cli-ergonomics`, `engineering-cargo-deps`, `engineering-perf`). Adapt to what's actually there.

## 3. Map work areas to specialists

One specialist per work area, usually. Combine where the project is too small to justify two distinct ones. Don't split where a single specialist handles a coherent surface (don't separate "API design" from "API implementation" — same person).

## 4. For each specialist, decide

```yaml
name: <discipline>-<role>          # kebab-case, namespaced
description: <action-oriented>     # this is what the orchestrator routes on
model: haiku | sonnet | opus
tools: <minimum set>
system_prompt:
  - mission (one sentence)
  - project docs to load by reference
  - invariants (verbatim from project CLAUDE.md)
  - skill-selection-at-runtime instruction
  - scope boundaries (what to handle, what to return to orchestrator)
```

## 5. Write files

Write each specialist as `.claude/agents/<discipline>/<name>.md` of the **consuming project's** working directory. Use this exact shape (no external template file required — compose it inline):

```markdown
---
name: <discipline>-<role>
description: <one sentence, third person, action oriented — what triggers this agent>
model: <haiku | sonnet | opus>
tools: <minimum comma-separated set>
---

You are the <role> specialist for the <discipline> chapter of this project.

# Mission

<one sentence>

# Project context to load

Before acting, read in parallel:
- `CLAUDE.md`
- <relevant project docs by path — e.g. `docs/architecture/01-database.md`>

# Invariants

Pulled from the project's `CLAUDE.md` — do not violate:
- <invariant 1>
- <invariant 2>

# Skill selection

Before acting, scan the skills available in your system prompt and invoke the most specific match via the Skill tool. Don't hardcode a skill list — choose per task.

# Scope

You handle:
- <list>

You don't handle (return to the orchestrator):
- <list>
```

Then write `.claude/agents/<discipline>/CHAPTER.md` — a roster table with one row per specialist (name, description, model, one-line example invocation), plus a "How to invoke this chapter" section pointing to `agentic-chapters:agent-orchestrator` for routing and `agentic-chapters:multi-agent-manager` for tuning.

Finally, append the delegation reminder to the project's `CLAUDE.md` using the exact block printed in `ai-agents-architect.md` step 6.

## 6. Mint pipelines (optional, if requested)

If the user asked for one or more pipelines alongside the chapter (e.g. "engineering with a feature-implement pipeline"), follow the `agentic-chapters:pipeline-design` skill for each pipeline. That skill defines the schema (phases, exit criteria, document handoffs, gates, iteration loops with caps, rules CR-*/AR-*, failure modes, push-back severity) and the design heuristics.

Constraints:
- Pipelines reference roster members by name. A pipeline can only reference specialists you've already minted in this chapter (or that exist in another chapter via `<chapter>/<specialist>` syntax). If a pipeline needs a specialist the roster doesn't have, mint that specialist first, then design the pipeline.
- Pipeline files live at `.claude/agents/<discipline>/pipelines/<pipeline-name>.md` of the consuming project. Names are kebab-case and discipline-prefixed (e.g. `engineering-feature-implement`, `devops-incident-response`).
- After writing each pipeline, update the chapter's `CHAPTER.md` to include a `## Pipelines` section listing the pipeline by name, with description and one-line example invocation (`/agentic-chapters:run-pipeline <discipline> <pipeline-name> "<example input>"`).

Skip this step if the user only asked for a roster.

## 7. Report back

Keep it tight:
- Roster table
- Model distribution (e.g. "1 Opus, 4 Sonnet, 2 Haiku")
- One-line example invocation per agent
- If pipelines were minted: pipeline names, phase counts, one-line example invocations
- Anything notable about the project survey that shaped the roster
