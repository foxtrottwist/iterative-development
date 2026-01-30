# Iterative Development

Development task orchestration using stateless iteration loops.

## What It Does

Orchestrates multi-task development by running each task as an **iteration loop**—multiple fresh-context passes until the task completes. Context accumulates in files, not conversation history. This prevents context degradation and enables reliable resumption.

```
RESUME?  ->  INTERVIEW  ->  PLAN  ->  LOOP  ->  CHECK  ->  WRAP
(check       (discover)    (tasks)  (iterate) (review)  (commit)
 state)
```

This builds on the Ralph Wiggum Technique (Geoffrey Huntley): instead of growing conversation history until context degrades, reset the context window every iteration. State lives in files. Each fresh agent reads those files to pick up where the last one left off.

## The Pattern

```
Main Session (orchestration)
├── Interview (gather requirements)
├── Plan (decompose into tasks)
├── For each task:
│   └── Iteration Loop ← Core pattern
│       ├── Iteration 1: Fresh agent reads state → works → commits
│       ├── Iteration 2: Fresh agent reads progress → continues
│       └── ... until DONE signal or max iterations
├── Review Loop (fresh iterations until PASS)
└── Wrap (final commit)
```

Each iteration spawns a **completely fresh sub-agent**. No memory of previous iterations except what's written to files. The progress file is how iterations communicate.

## Attribution

**The Ralph Wiggum Technique** — Geoffrey Huntley coined this pattern: a loop that repeatedly invokes an agent with a prompt file, allowing iterative refinement until completion. Instead of growing conversation history until context degrades, reset the context window every iteration. State lives in files. Each fresh agent reads those files to pick up where the last one left off.

**Anthropic's Research** — Documentation on long-running agents, sub-agent coordination, and the Task tool informed the dispatch mechanics.

This skill assembles these patterns into a cohesive workflow. The interview phase, guardrail accumulation, and task decomposition structure are adaptations of common practices.

## Installation

Copy the `iterative-development` directory to your Claude skills location:

```bash
cp -r iterative-development /path/to/skills/user/
```

Or reference SKILL.md directly in your Claude configuration.

## Usage

### Start a New Feature

```
/iter-dev implement user authentication with email/password
```

The skill will:
1. Interview you to clarify requirements
2. Decompose into tasks with iteration limits
3. Run each task as an iteration loop
4. Review with fresh iterations until PASS
5. Wrap and commit

### Resume After Interruption

```
/iter-dev resume
```

Reads state from `.claude/iterative-dev/{feature}/` and continues at the exact iteration.

### Check Status

```
/iter-dev status
```

Shows task progress with iteration counts.

## How It Works

### The Iteration Loop (Per Task)

Each task runs its own iteration loop:

```
Task T1: "Create User model"
├── Iteration 1: Fresh agent creates file skeleton
├── Iteration 2: Fresh agent adds properties
├── Iteration 3: Fresh agent completes criteria → T1_DONE
└── Move to T2
```

Every iteration:
1. Spawn fresh sub-agent (no memory)
2. Agent reads progress.md, guardrails.md, context.md
3. Agent continues from where last iteration stopped
4. Agent commits work, appends to progress.md
5. If criteria met: emit completion signal
6. If not done: exit, next iteration continues

### State Files

All state lives in `.claude/iterative-dev/{feature-slug}/`:

- `state.json` — Phase, current task, iteration count
- `prd.md` — Requirements from interview
- `tasks.md` — Task breakdown with max iterations
- `context.md` — Brief summary for sub-agents
- `progress.md` — Iteration log (critical for the loop)
- `guardrails.md` — Accumulated lessons

### Progress File is Critical

The progress file bridges context windows:

```markdown
## T1 - Iteration 1 - {timestamp}
**Did:** Created User.swift, added skeleton
**Remaining:** Add properties, Codable conformance
**Commit:** abc1234

## T1 - Iteration 2 - {timestamp}
**Did:** Added all properties and Codable
**Remaining:** SwiftData annotations
**Commit:** def5678

## T1 - Iteration 3 - {timestamp}
**Did:** Added @Model, all criteria met
**Remaining:** None
**Signal:** T1_DONE
**Commit:** ghi9012
```

Each iteration reads this to understand current state, then appends its own entry.

### Model Selection

Tasks are assigned to appropriate models:

| Task Type | Model | Max Iterations |
|-----------|-------|----------------|
| Simple file ops | haiku | 3-5 |
| Standard implementation | sonnet | 5-10 |
| Complex debugging/architecture | opus | 10-15 |

## Directory Structure

```
iterative-development/
├── SKILL.md              # Main skill definition
├── README.md             # This file
├── LICENSE               # MIT
├── agents/               # Sub-agent definitions
│   ├── impl-haiku.md     # Simple tasks
│   ├── impl-sonnet.md    # Standard tasks
│   ├── impl-opus.md      # Complex tasks
│   ├── review.md         # Code review loop
│   ├── fix.md            # Issue fix loop
│   └── test.md           # Test generation loop
├── references/           # Supporting documentation
│   ├── interview.md      # Interview templates
│   ├── tasks.md          # Task format guide
│   ├── prompts.md        # Iteration prompt templates
│   └── state.md          # State file schemas
└── .github/
    └── workflows/
        └── release.yml   # Packaging workflow
```

## Anti-Patterns

- **Carrying context** — Passing previous iteration output into prompts defeats the purpose
- **Skipping progress updates** — Next iteration has no idea what happened
- **Ignoring guardrails** — Repeat mistakes across iterations
- **Too few iterations** — Complex tasks need room to converge
- **Too many iterations** — Thrashing without progress wastes resources
- **Giant tasks** — Let iterations handle complexity, but scope reasonably

## Key Differences from Single-Agent Approach

| Single Agent | Iterative Development |
|--------------|------------------------|
| Context grows with conversation | Fresh context every iteration |
| Eventually hits context degradation | Clean slate each time |
| Hard to resume after interruption | Resume at exact iteration |
| Mistakes compound | Each iteration reads clean state |
| Complex prompts try to cover everything | Simple prompts, agents read files |

## License

MIT
