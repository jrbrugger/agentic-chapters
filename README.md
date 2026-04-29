# agentic-chapters

A Claude Code plugin for spinning up discipline-specific "chapters" of specialist agents — and optional **playbooks** the chapter runs against repeatable kinds of work.

You name a discipline (engineering, marketing, research, ops, design, security, anything). The architect surveys your project, decides which specialists are needed for *this* codebase, picks the right model for each (lean: Haiku for narrow lookup/check work, Sonnet for reasoning, Opus only for top-level design), and writes each agent file into your project. If you ask for one, the architect also mints a **playbook** — a named, phase-structured workflow with document handoffs, exit criteria, iteration bounds, severity-tiered push-back, and human gates — modeled after [Kaelig's design-system component playbook](https://www.kaelig.fr/design-system-components-with-ai-agent-teams/) but generalized to any discipline.

At runtime, an orchestrator routes ad-hoc tasks across the chapter or executes named playbooks end-to-end; a tuning manager keeps both the roster and the playbooks lean over time.

## What's in the box

| Agent | Model | Role |
|---|---|---|
| `ai-agents-architect` | opus | Surveys the project, designs a chapter, writes specialist files |
| `agent-orchestrator` | sonnet | Routes tasks at runtime — fans out parallel, sequences dependencies, synthesizes |
| `multi-agent-manager` | sonnet | Audits an existing chapter and tunes it — overlap, dead weight, model mismatch, prompt drift |

Plus six slash commands (namespaced):
- `/agentic-chapters:new-chapter` — mint a chapter (and optional playbooks).
- `/agentic-chapters:chapter-route` — route an ad-hoc task to the chapter.
- `/agentic-chapters:chapter-tune` — audit the chapter (roster + playbooks).
- `/agentic-chapters:run-playbook` — execute a named playbook end-to-end.
- `/agentic-chapters:playbook-status` — inspect a running or completed playbook run.
- `/agentic-chapters:playbook-resume` — resume a halted playbook run.

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

The manager audits the chapter — overlapping descriptions, agents never invoked, oversized models, drifted prompts, over-broad tool permissions, plus playbook-level findings (phase bloat, dead phases, unbounded loops, rubber-stamped gates) — proposes changes, applies on approval.

## Playbooks

A **playbook** is a named, phase-structured workflow within a chapter. Where the chapter is the *team*, the playbook is the *playbook* the team runs against a specific kind of work. Playbooks are optional and additive — a chapter can have zero, one, or many.

### Mint a playbook alongside a chapter

```
/agentic-chapters:new-chapter engineering "with a feature-implement playbook"
```

The architect designs the roster (as before) and also writes `.claude/agents/engineering/playbooks/engineering-feature-implement.md` following the universal playbook schema.

### Run a playbook

```
/agentic-chapters:run-playbook engineering engineering-feature-implement "add a /healthz endpoint"
```

The orchestrator walks phases in order, dispatches specialists per phase (parallel or sequential), writes per-specialist handoff artifacts to `.claude/playbook-runs/<run-id>/`, enforces exit criteria, respects iteration loop caps, and surfaces any `[BLOCKING]` issues to you as gates.

Add `--dry-run` to see the plan without invoking specialists:
```
/agentic-chapters:run-playbook engineering engineering-feature-implement "add /healthz" --dry-run
```

### Inspect or resume a run

```
/agentic-chapters:playbook-status                     # list recent runs
/agentic-chapters:playbook-status 20260429-143022-feature-implement
/agentic-chapters:playbook-resume 20260429-143022-feature-implement
```

### The universal playbook schema

Every playbook file has the same shape (full spec in the `agentic-chapters:playbook-design` skill). The seven load-bearing pieces:

1. **Phases** — ordered list, each with observable exit criteria.
2. **Specialists per phase** — referenced by name; same chapter (`engineering-spec-author`) or cross-chapter (`marketing/copy-reviewer`); parallel or sequential.
3. **Document handoffs** — per-specialist artifact files, no clobbering between specialists in the same phase.
4. **Iteration loops** — soft cap that escalates to a `[BLOCKING]` gate (never silent abort).
5. **Gates** — human checkpoints with explicit options menus.
6. **Rules** — mandatory (CR-*) vs advisory (AR-*).
7. **Failure modes** — what happens when an iteration cap is hit, a specialist returns no artifact, a gate is rejected.

### Cross-discipline applicability

Same grammar, different cast:

| Discipline | Example playbook | Phases |
|---|---|---|
| Engineering | `feature-implement` | Spec → Implement → Test → Review |
| Frontend / Design system | `component-build` | Understand (Figma, lib, arch) → Build (code, a11y, stories) → Verify (visual, quality) |
| Marketing | `launch-campaign` | Research → Build (copy, assets, channels) → Verify (review, A/B plan) |
| Business validation | `validate-idea` | Research → Build (unit econ, risks) → Verify (go/no-go gate) |
| DevOps | `incident-response` | Detect → Contain → Diagnose → Remediate → Postmortem |
| Security | `threat-model-feature` | Inventory → STRIDE → Mitigations → Review |

The plugin doesn't ship any starter playbooks — that would hardcode disciplines, which contradicts the meta-design. Playbooks are minted into your project by the architect.

## Design principles

- **Chapters live in your project, not in this plugin.** Only the three executives ship in this repo. Specialists are project-specific and get written into the consuming project's `.claude/agents/<discipline>/`. That's why the same plugin works for an engineering chapter on a Next.js app and a research chapter on a data playbook.

- **Skill selection is per-task, per-agent.** Specialist prompts don't hardcode skills; they pick at runtime from whatever skill library is available. As your skill ecosystem grows, agents adapt without rewrites.

- **Models are picked per agent, leaning cheap.** Haiku for narrow lookup/check/listing/format work, Sonnet for multi-step reasoning (implementation, refactor, review, debug), Opus only for top-level architect-class roles. A healthy chapter typically lands at ~3–5 Sonnet + 2–4 Haiku, *not* an all-Sonnet roster. The manager downgrades any agent with unused headroom during tuning.

- **Descriptions are the contract.** Names drift; descriptions are how the orchestrator routes. The architect spends real effort on them; the manager keeps them sharp over time.

- **Specialists reference project docs, they don't copy them.** A specialist's system prompt says "read `docs/architecture/01-database.md`" rather than embedding the contents. Prompts stay short; updates to the docs propagate automatically.

## Repo layout

```
agentic-chapters/
├── .claude-plugin/plugin.json
├── agents/
│   ├── ai-agents-architect.md         # opus — designs chapters and playbooks
│   ├── agent-orchestrator.md          # sonnet — routes tasks; runs playbooks
│   └── multi-agent-manager.md         # sonnet — audits chapters and playbooks
├── skills/
│   ├── chapter-bootstrap/SKILL.md     # recipe for chapter scaffolding
│   └── playbook-design/SKILL.md       # recipe + schema for playbooks
├── commands/
│   ├── new-chapter.md                 # /agentic-chapters:new-chapter
│   ├── chapter-route.md               # /agentic-chapters:chapter-route
│   ├── chapter-tune.md                # /agentic-chapters:chapter-tune
│   ├── run-playbook.md                # /agentic-chapters:run-playbook
│   ├── playbook-status.md             # /agentic-chapters:playbook-status
│   └── playbook-resume.md             # /agentic-chapters:playbook-resume
└── README.md
```

The specialist-agent template, the playbook file schema, and the consuming-project `CLAUDE.md` fragment are inlined into the architect's prompt and the two skills — no external template files to drift out of sync.

## License

MIT — see `LICENSE`.
