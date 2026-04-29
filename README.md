# agentic-chapters

A Claude Code plugin for spinning up discipline-specific "chapters" of specialist agents.

You name a discipline — engineering, marketing, research, ops, design, anything. The architect surveys your project, decides which specialists are needed for *this* codebase, picks the right model for each (lean: Haiku for narrow lookup/check work, Sonnet for reasoning, Opus only for top-level design), and writes each agent file into your project. A runtime orchestrator routes incoming tasks across the chapter; a tuning manager keeps the roster lean over time.

## What's in the box

| Agent | Model | Role |
|---|---|---|
| `ai-agents-architect` | opus | Surveys the project, designs a chapter, writes specialist files |
| `agent-orchestrator` | sonnet | Routes tasks at runtime — fans out parallel, sequences dependencies, synthesizes |
| `multi-agent-manager` | sonnet | Audits an existing chapter and tunes it — overlap, dead weight, model mismatch, prompt drift |

Plus three slash commands (namespaced): `/agentic-chapters:new-chapter`, `/agentic-chapters:chapter-route`, `/agentic-chapters:chapter-tune`.

## Install

### From a local clone (recommended for now)

Clone this repo, then load it on every Claude Code session with the `--plugin-dir` flag from any project you want to use it in:

```
git clone https://github.com/jrbrugger/agentic-chapters.git ~/Documents/Apps/agentic-chapters
cd /path/to/some/project
claude --plugin-dir ~/Documents/Apps/agentic-chapters
```

### Via a Claude Code marketplace

If you've added this repo to a marketplace (`/plugin marketplace add <url>`), then:

```
/plugin install agentic-chapters
```

After install, run `/reload-plugins` if Claude Code doesn't pick it up automatically.

## Usage

> Plugin slash commands and agents are namespaced. Invoke as `/agentic-chapters:<command>` or use the `subagent_type: "agentic-chapters:<agent>"` form when calling via `Agent`.

### Stand up a chapter

```
/agentic-chapters:new-chapter engineering "Next.js 16 + Supabase + Stripe + Tailwind v4"
```

The architect surveys your repo (CLAUDE.md, package.json, docs/), hires 4–10 specialists tailored to your stack, picks Haiku/Sonnet/Opus per agent (leaning cheap — see *Design principles*), and writes them to `.claude/agents/engineering/` of the **current project**. Each gets a sharp description so the orchestrator can route on it. The architect also appends a short delegation reminder to the project's `CLAUDE.md`.

### Route a task

```
/agentic-chapters:chapter-route engineering "refactor the pricing engine to handle multi-currency"
```

The orchestrator decomposes the task, fans out to relevant specialists in parallel, sequences anything that depends on prior output, and returns a single synthesized result. You don't see the orchestration log — just the answer.

### Tune a chapter

```
/agentic-chapters:chapter-tune engineering
```

The manager audits the chapter — overlapping descriptions, agents never invoked, oversized models, drifted prompts, over-broad tool permissions — proposes changes, applies on approval.

## Design principles

- **Chapters live in your project, not in this plugin.** Only the three executives ship in this repo. Specialists are project-specific and get written into the consuming project's `.claude/agents/<discipline>/`. That's why the same plugin works for an engineering chapter on a Next.js app and a research chapter on a data pipeline.

- **Skill selection is per-task, per-agent.** Specialist prompts don't hardcode skills; they pick at runtime from whatever skill library is available. As your skill ecosystem grows, agents adapt without rewrites.

- **Models are picked per agent, leaning cheap.** Haiku for narrow lookup/check/listing/format work, Sonnet for multi-step reasoning (implementation, refactor, review, debug), Opus only for top-level architect-class roles. A healthy chapter typically lands at ~3–5 Sonnet + 2–4 Haiku, *not* an all-Sonnet roster. The manager downgrades any agent with unused headroom during tuning.

- **Descriptions are the contract.** Names drift; descriptions are how the orchestrator routes. The architect spends real effort on them; the manager keeps them sharp over time.

- **Specialists reference project docs, they don't copy them.** A specialist's system prompt says "read `docs/architecture/01-database.md`" rather than embedding the contents. Prompts stay short; updates to the docs propagate automatically.

## Repo layout

```
agentic-chapters/
├── .claude-plugin/plugin.json
├── agents/
│   ├── ai-agents-architect.md         # opus — designs chapters
│   ├── agent-orchestrator.md          # sonnet — routes tasks
│   └── multi-agent-manager.md         # sonnet — tunes chapters
├── skills/
│   └── chapter-bootstrap/SKILL.md     # bootstrap recipe (inlined templates)
├── commands/
│   ├── new-chapter.md                 # /agentic-chapters:new-chapter
│   ├── chapter-route.md               # /agentic-chapters:chapter-route
│   └── chapter-tune.md                # /agentic-chapters:chapter-tune
└── README.md
```

The specialist-agent template and the consuming-project `CLAUDE.md` fragment are inlined into the architect's prompt and the bootstrap skill — no external template files to drift out of sync.

## License

MIT — see `LICENSE`.
