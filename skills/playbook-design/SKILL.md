---
name: playbook-design
description: Recipe the ai-agents-architect follows to mint a named playbook within a chapter. Defines the playbook file schema (phases, exit criteria, document handoffs, gates, iteration loops with bounds, severity protocol, rules, failure modes), the design heuristics that come from a real-world agentic playbook pattern, and the don'ts. Invoke when designing any new playbook.
---

# Playbook design recipe

A **playbook** is a named, phase-structured workflow within a chapter. Where the chapter is the *team*, the playbook is the *playbook* the team runs against a specific kind of work. A chapter can have zero, one, or many playbooks — they're additive to the roster, never required.

A playbook file lives at `.claude/agents/<discipline>/playbooks/<playbook-name>.md` of the consuming project.

## When to mint a playbook (vs. leaving routing ad-hoc)

Mint a playbook when **all** of these hold:

- The work has a **repeatable shape** — multiple invocations follow the same phase structure (e.g. *every* component build goes through Understand → Build → Verify; *every* incident goes through Detect → Contain → Diagnose → Remediate → Postmortem).
- The phases have **distinct, observable exit criteria** — you can tell when a phase is done, not just guess.
- There's value in **document-mediated handoffs** — later phases benefit from a structured artifact written by earlier phases (a brief, an architecture decision, a remediation plan), not just message-passing.
- There are **gate decisions** that should always pause for a human, or there's an **iteration loop** that needs a hard cap.

If only one or two hold, leave the work to the orchestrator's ad-hoc routing. Playbooks have overhead — phase boundaries, artifact files, state tracking — that pays back only when the structure is reused.

## The playbook file schema

YAML frontmatter for the structured spine, markdown body for the prose contracts:

```markdown
---
name: <playbook-name>                       # discipline-prefixed kebab-case, e.g. "engineering-feature-implement"
discipline: <chapter-name>
version: <semver>                           # bump on revision so the manager can detect stale state
description: <one sentence — what this playbook does end-to-end>
run_artifacts_dir: .claude/playbook-runs/   # default; override per playbook if you need a different audit trail location
phases:                                     # ordered index of phase names (the body holds the prose per phase)
  - <phase-1-name>
  - <phase-2-name>
  - <phase-3-name>
---

# Playbook: <name>

## Phase: <phase-1-name>

- **Specialists** (parallel | sequential): `<specialist-1>`, `<specialist-2>`
- **Reads**: `<input-path>` (e.g. the user's prompt, prior phase artifacts at `<run-id>/<prior-phase>/<artifact>.md`)
- **Writes** (per specialist, no clobbering):
  - `<run-id>/<this-phase>/<specialist-1>.md` — required fields: `<list>`
  - `<run-id>/<this-phase>/<specialist-2>.md` — required fields: `<list>`
- **Exit criteria** (must all pass before next phase begins):
  - <criterion 1 — observable, not "looks good">
  - <criterion 2>
- **Loops** (optional, scoped to this phase):
  - `<executor>` ↔ `<reviewer>`, soft cap N
  - Exit when: <criterion>
- **Gate at exit** (optional):
  - Trigger: any specialist returns `[BLOCKING]`, OR <other condition>
  - Human decision: <options menu — "proceed", "rerun phase", "edit artifact and continue">

(repeat per phase)

## Rules

### Mandatory (CR-*)
- CR-1: <rule that must hold for the playbook's output to be valid>
- CR-2: <rule>

### Advisory (AR-*)
- AR-1: <rule the specialists should respect but won't halt the playbook if violated>

## Failure modes

- **Iteration cap exhaustion** → escalate to a synthetic `[BLOCKING]` gate. Do not abort silently.
- **`[BLOCKING]` mid-loop** → halt the loop immediately, do not count toward the iteration cap, surface to human.
- **Specialist returns no artifact** → mark phase failed, halt with reason in `state.json`.
- **Gate rejection** → rerun prior phase by default. Override here if a specific phase should rewind further or abort.

## Push-back severity

- `[BLOCKING]` — halt the playbook (or the loop), surface to human.
- `[CONCERN]` — note in the handoff artifact, continue to next phase.
- `[SUGGESTION]` — note for retrospective, continue.
```

## Field-by-field guidance

### `phases`

The ordered index. Names are short, kebab-case verbs or nouns describing the phase's *output*: `understand`, `build`, `verify`, `detect`, `contain`, `diagnose`, `remediate`, `postmortem`. Don't name phases after agents (`spec-writer`) or steps (`step-1`).

Three phases is the most common shape (Understand → Build → Verify). Five-phase incident response is the next most common. More than seven phases is almost always phase bloat — you're describing tasks, not phases.

### Specialists per phase

Reference roster members by name. Same chapter: bare name (`engineering-spec-author`). Cross-chapter: `<chapter>/<specialist>` (e.g. `marketing/copy-reviewer`). Cross-chapter references are first-class — playbooks can pull a specialist from another chapter when the work crosses disciplines.

State whether specialists run **parallel** (independent — multiple reviewers, multiple analyzers) or **sequential** (executor → reviewer where the reviewer needs the executor's output). Don't leave this implicit.

### Reads / Writes (the document handoff contract)

This is the most important part of the schema. Phases communicate through artifact files, not through the orchestrator's message-passing. Each specialist writes to a per-specialist file (e.g. `<run-id>/build/code-writer.md` and `<run-id>/build/a11y-auditor.md` are separate) — never a shared file that two specialists can clobber.

Each `Writes` entry should list **required fields** the artifact must contain. The next phase's specialists rely on those fields existing — if they don't, the next phase fails fast rather than improvising.

### Exit criteria

Must be **observable**. "TypeScript compiles," "lint passes," "all CR-* rules cited in artifact," "human approved at gate" — these are observable. "Quality is good," "design feels right" — these are not. The orchestrator checks exit criteria at phase end; if any fail, the phase doesn't advance.

### Loops

Loops are scoped to a single phase. They model executor ↔ reviewer cycles within a phase (e.g. Code Writer ↔ Visual Reviewer in the article). Always have a **soft cap** (e.g. 5). At cap, escalate to a synthetic `[BLOCKING]` gate — don't abort. Soft caps preserve prior work; hard caps waste it.

### Gates

A gate is a human checkpoint, usually at phase exit. Specify the **trigger** (what causes the gate to fire — typically a `[BLOCKING]` from a specialist) and the **human decision** (what options the orchestrator surfaces). Don't put a gate at every phase exit by reflex; gates are friction. Use them for genuine human-decision points: "is this architecture right?", "should we ship despite the failing test?", "is this remediation acceptable?".

### Rules: CR-* vs AR-*

- **Mandatory (CR-*)** — must hold for the playbook's output to be valid. Specialists cite the CR-* numbers they depend on or violate. Verify-phase specialists check CR-* compliance.
- **Advisory (AR-*)** — should hold; if violated, the specialist notes it as a `[SUGGESTION]` or `[CONCERN]` in their artifact.

This is from the article's pattern (Component Rules / Advisory Rules). Encode rules at the persistence layer where they make sense: hard rules in CR-*, soft preferences in AR-*.

### Failure modes

These are the *invariants* the orchestrator enforces. Without them spelled out, the orchestrator improvises and you get inconsistent behavior across runs. Spell them out even when they're "obvious".

## Design heuristics (mapped to the article's seven transferable principles)

When designing a playbook, lean on these:

1. **Separate creation from evaluation.** Different specialists for build vs. verify. Don't ask the Code Writer to also be the Quality Gate.
2. **Front-load hard questions.** Use a phase-1 gate (architecture review, scope confirmation) so blocking questions surface before downstream work is paid for.
3. **Encode rules at the right persistence layer.** CR-* in this file (load-bearing). AR-* in this file (preferences). Discipline conventions in `CLAUDE.md`. Project tokens in code.
4. **Define exit criteria upfront.** A phase without exit criteria is not a phase — it's a mood.
5. **Design human gates as features, not interruptions.** A good gate has a clear options menu and a clear default. A bad gate just says "approve y/n?" and erodes trust.
6. **Rules compound, but failures evolve.** Add new CR-* and AR-* sparingly. Update failure modes whenever a real run reveals a new edge case the orchestrator didn't handle.
7. **Be transparent about limitations.** If a phase can't reliably catch a class of issue (e.g. visual regressions Visual Reviewer misses), say so in the phase's body. Don't pretend the playbook is infallible.

## Common playbook patterns

### 3-phase Understand → Build → Verify

The article's design-system component playbook. Generalizes to: any task where you research/design first, build second, validate third. Engineering features, marketing campaigns, security audits.

### 5-phase incident response

Detect → Contain → Diagnose → Remediate → Postmortem. Distinct from the 3-phase pattern because each phase has a different *time pressure* — Detect/Contain are minutes; Diagnose is hours; Remediate is days; Postmortem is the week after. Playbook encodes the temporal shape.

### 4-phase validate-build

Validate (problem, market, unit economics) → Plan (scope, milestones, team) → Build → Verify. For business-validation and product-launch playbooks.

### N-phase playbooks

Rare. Most "long" playbooks decompose into nested 3-phase playbooks or are over-described. If you find yourself writing a 9-phase playbook, ask whether phases 4–7 are actually one Build phase in disguise.

## Don't

- **Don't mint phases without observable exit criteria.** A phase you can't verify is done isn't a phase, it's a vibe. Either define the criterion or merge with the next phase.
- **Don't loop without a cap.** Every loop must have a soft cap that escalates to a gate. Unbounded loops are how playbooks burn down a token budget overnight.
- **Don't put gates at every phase boundary by reflex.** Gates are friction. Use them where genuine human judgment is required, not as decoration.
- **Don't share artifact files between specialists in the same phase.** Per-specialist files only. If you need a stitched view, the orchestrator stitches at phase exit.
- **Don't bake project specifics into the playbook.** A playbook is a *shape*. Specifics (file paths, library names, token sets) belong in the specialists' system prompts or in `CLAUDE.md`. If the playbook file mentions `next.config.ts`, you're making it brittle.
- **Don't create a playbook for a one-off task.** If you'd run this exactly once, route ad-hoc. Playbooks pay off when reused.
- **Don't reference specialists that don't exist in the chapter's roster.** The architect should mint missing specialists *first*, then design the playbook. The orchestrator can't dispatch agents that don't exist.
