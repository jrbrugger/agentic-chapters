# agentic-chapters

A Claude Code plugin for spinning up discipline-specific "chapters" of specialist agents.

You name a discipline — engineering, marketing, research, ops, design, anything. The architect surveys your project, decides which specialists are needed for *this* codebase, picks the right model for each (Haiku for narrow lookups, Sonnet for most work, Opus for deep design), and writes each agent file into your project. A runtime orchestrator routes incoming tasks across the chapter; a tuning manager keeps the roster lean over time.

## What's in the box

| Agent | Model | Role |
|---|---|---|
| `ai-agents-architect` | opus | Surveys the project, designs a chapter, writes specialist files |
| `agent-orchestrator` | sonnet | Routes tasks at runtime — fans out parallel, sequences dependencies, synthesizes |
| `multi-agent-manager` | sonnet | Audits an existing chapter and tunes it — overlap, dead weight, model mismatch, prompt drift |

Plus three slash commands: `/new-chapter`, `/chapter-route`, `/chapter-tune`.

## Install

### From GitHub (once published)

```
/plugin install <github-org>/agentic-chapters
```

### Local development

```
/plugin marketplace add ~/Documents/Apps/agentic-chapters
/plugin install agentic-chapters
```

## Usage

### Stand up a chapter

```
/new-chapter engineering "Next.js 16 + Supabase + Stripe + Tailwind v4"
```

The architect surveys your repo (CLAUDE.md, package.json, docs/), hires 4–10 specialists tailored to your stack, picks Haiku/Sonnet/Opus per agent based on task shape, and writes them to `.claude/agents/engineering/`. Each gets a sharp description so the orchestrator can route on it.

### Route a task

```
/chapter-route engineering "refactor the pricing engine to handle multi-currency"
```

The orchestrator decomposes the task, fans out to relevant specialists in parallel, sequences anything that depends on prior output, and returns a single synthesized result. You don't see the orchestration log — just the answer.

### Tune a chapter

```
/chapter-tune engineering
```

The manager audits the chapter — overlapping descriptions, agents never invoked, oversized models, drifted prompts, over-broad tool permissions — proposes changes, applies on approval.

## Design principles

- **Chapters live in your project, not in this plugin.** Only the three executives ship in this repo. Specialists are project-specific and get written into the consuming project's `.claude/agents/<discipline>/`. That's why the same plugin works for an engineering chapter on a Next.js app and a research chapter on a data pipeline.

- **Skill selection is per-task, per-agent.** Specialist prompts don't hardcode skills; they pick at runtime from whatever skill library is available. As your skill ecosystem grows, agents adapt without rewrites.

- **Models are picked per agent, not globally.** The architect decides Haiku/Sonnet/Opus based on the work shape. Most specialists are Sonnet. Routine lookup work goes to Haiku. Architect-class agents go to Opus.

- **Descriptions are the contract.** Names drift; descriptions are how the orchestrator routes. The architect spends real effort on them; the manager keeps them sharp over time.

- **Specialists reference project docs, they don't copy them.** A specialist's system prompt says "read `docs/architecture/01-database.md`" rather than embedding the contents. Prompts stay short; updates to the docs propagate automatically.

## Repo layout

```
agentic-chapters/
├── .claude-plugin/plugin.json
├── agents/
│   ├── ai-agents-architect.md         # opus — designs
│   ├── agent-orchestrator.md          # sonnet — routes
│   └── multi-agent-manager.md         # sonnet — tunes
├── skills/
│   └── chapter-bootstrap/SKILL.md     # the recipe the architect follows
├── commands/
│   ├── new-chapter.md                 # /new-chapter
│   ├── chapter-route.md               # /chapter-route
│   └── chapter-tune.md                # /chapter-tune
├── templates/
│   ├── chapter-agent.template.md      # starting point for specialist files
│   └── chapter-claude.md.fragment     # appended to consuming-project CLAUDE.md
└── README.md
```

## License

MIT — see `LICENSE`.
