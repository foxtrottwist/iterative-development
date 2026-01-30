# Iterative Work

Knowledge work orchestration using stateless iteration loops.

## What It Does

Orchestrates multi-phase knowledge work by running each phase as an **iteration loop**—multiple fresh-context passes until the phase completes. Context accumulates in files, not conversation history. This prevents context degradation and enables reliable resumption.

```
RESUME?  →  DISCOVER  →  PLAN  →  EXECUTE  →  VERIFY  →  DELIVER
(check      (clarify     (phases)  (iterate)  (review)  (package)
 state)      scope)
```

This builds on the Ralph Wiggum Technique (Geoffrey Huntley): instead of growing conversation history until context degrades, reset the context window every iteration. State lives in files. Each fresh agent reads those files to pick up where the last one left off.

## The Pattern

```
Main Session (orchestration)
├── Discover (gather requirements)
├── Plan (decompose into phases)
├── For each phase:
│   └── Iteration Loop ← Core pattern
│       ├── Pass 1: Fresh context reads state → works → saves
│       ├── Pass 2: Fresh context reads progress → continues
│       └── ... until DONE signal or max iterations
├── Verify Loop (fresh passes until PASS)
└── Deliver (package outputs)
```

Each iteration spawns a **completely fresh context**. No memory of previous passes except what's written to files. The progress file is how iterations communicate.

## Four Domains

Handles four knowledge work domains, each with a tailored phase pattern:

| Domain | Phases | Use When |
|--------|--------|----------|
| Research | R1→R2→R3→R4 | Source discovery → evaluation → patterns → synthesis |
| Writing | D1→D2→D3→D4 | Structure → draft → revise → polish |
| Analysis | A1→A2→A3→A4 | Gather → patterns → interpret → recommend |
| Planning | P1→P2→P3→P4 | Context → options → evaluate → document |

Phases can combine across domains: research feeding into writing (R1-R4 then D1-D4), or analysis leading to planning (A1-A4 then P1-P4).

## Attribution

**The Ralph Wiggum Technique** — Geoffrey Huntley coined this pattern: a loop that repeatedly invokes an agent with a prompt file, allowing iterative refinement until completion. Instead of growing conversation history until context degrades, reset the context window every iteration. State lives in files. Each fresh agent reads those files to pick up where the last one left off.

**Iterative Development** — The development-focused sibling skill that applies the same pattern to coding tasks.

**Anthropic's Research** — Documentation on long-running agents and sub-agent coordination informed the architecture.

## Installation

Copy the `iterative-work` directory to your Claude skills location:

```bash
cp -r iterative-work /path/to/skills/user/
```

Or reference SKILL.md directly in your Claude configuration.

## Usage

### Start a New Task

```
/iter-work research the current state of AI governance frameworks
```

The skill will:
1. Interview you to clarify scope and deliverables
2. Decompose into phases with iteration limits
3. Run each phase as an iteration loop
4. Review with fresh passes until PASS
5. Package and deliver outputs

### Resume After Interruption

```
/iter-work resume
```

Reads state from `.iterative-work/{task-slug}/` and continues at the exact iteration.

### Check Status

```
/iter-work status
```

Shows phase progress with iteration counts.

## How It Works

### The Iteration Loop (Per Phase)

Each phase runs its own iteration loop:

```
Phase R2: "Source Evaluation"
├── Pass 1: Fresh context creates evaluation template
├── Pass 2: Fresh context evaluates first batch
├── Pass 3: Fresh context completes evaluations → R2_DONE
└── Move to R3
```

Every iteration:
1. Fresh context starts (no memory)
2. Reads progress.md, guardrails.md, context.md
3. Continues from where last iteration stopped
4. Saves work, appends to progress.md
5. If criteria met: emit completion signal
6. If not done: exit, next iteration continues

### State Files

All state lives in `.iterative-work/{task-slug}/`:

- `state.json` — Phase, current step, iteration count
- `brief.md` — Requirements from discovery
- `plan.md` — Phase breakdown with max iterations
- `context.md` — Brief summary for iterations
- `progress.md` — Iteration log (critical for the loop)
- `guardrails.md` — Accumulated lessons
- `sources/` — Research inputs
- `outputs/` — Deliverables

### Progress File is Critical

The progress file bridges context windows:

```markdown
## R2 - Iteration 1 - 2026-01-16T10:00:00
**Did:** Evaluated 4 of 12 sources, created evaluation template
**Remaining:** 8 sources, synthesize scores
**Blockers:** None

## R2 - Iteration 2 - 2026-01-16T10:15:00
**Did:** Completed remaining evaluations
**Remaining:** None
**Signal:** R2_DONE
```

Each iteration reads this to understand current state, then appends its own entry.

### Iteration Limits

| Phase Complexity | Max Iterations | Characteristics |
|------------------|----------------|-----------------|
| Simple | 3 | Clear criteria, single output |
| Medium | 5 | Some judgment, multiple aspects |
| Complex | 8 | Synthesis required, quality bar |

## Directory Structure

```
iterative-work/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT
├── agents/               # Phase-specific guidance
│   ├── research.md       # R1-R4 research phases
│   ├── draft.md          # D1-D4 writing phases
│   ├── analyze.md        # A1-A4 analysis phases
│   ├── plan.md           # P1-P4 planning phases
│   ├── review.md         # Review loop
│   └── revise.md         # Revision loop
├── references/           # Supporting documentation
│   ├── interview.md      # Interview templates
│   ├── domains.md        # Domain phase patterns
│   ├── prompts.md        # Iteration prompt templates
│   └── state.md          # State file schemas
└── .github/
    └── workflows/
        └── release.yml   # Packaging workflow
```

## Anti-Patterns

- **Skipping progress updates** — Next iteration has no idea what happened
- **Ignoring guardrails** — Repeat mistakes across iterations
- **Giant phases** — Let iterations handle complexity, but scope reasonably
- **Too few iterations** — Complex work needs room to converge
- **Too many iterations** — Thrashing without progress wastes resources

## Key Differences from Single-Pass Approach

| Single Pass | Iterative Work |
|-------------|----------------|
| Context grows with conversation | Fresh context every iteration |
| Eventually hits context degradation | Clean slate each time |
| Hard to resume after interruption | Resume at exact iteration |
| Mistakes compound | Each iteration reads clean state |
| Complex prompts try to cover everything | Simple prompts, agents read files |

## License

MIT
