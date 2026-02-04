---
name: ralph-plan
description: Scaffold and plan a Ralph Wiggum autonomous loop for this project. Sets up PROMPT_plan.md, PROMPT_build.md, AGENTS.md, IMPLEMENTATION_PLAN.md, and loop.sh — then runs the planning phase to analyze gaps between specs and code.
allowed-tools: Read, Grep, Glob, Write, Edit, Bash, Task, AskUserQuestion
argument-hint: [goal-description]
---

# Ralph Plan — Set Up and Plan a Ralph Wiggum Loop

You are setting up a Ralph Wiggum autonomous coding loop for this project. Ralph is an iterative AI development methodology where the same prompt is fed to Claude repeatedly in a bash loop. Each iteration gets a fresh context window, reads the persistent IMPLEMENTATION_PLAN.md from disk, picks the most important task, implements it, commits, and exits. The loop restarts with fresh context.

## Phase 1: Study the Project

Before scaffolding anything, understand the project:

1. Use up to 20 parallel Sonnet subagents to study the codebase structure — find the source directories, test directories, config files, build system, and existing patterns.
2. Identify: programming language(s), framework(s), package manager, test runner, linter, type checker, build commands.
3. Check if any Ralph files already exist (PROMPT_plan.md, PROMPT_build.md, AGENTS.md, IMPLEMENTATION_PLAN.md, loop.sh). If they do, read them and ask the user if they want to overwrite or update.
4. Check if a `specs/` directory exists. If it does, read existing specs. If not, note this — we will create it in Phase 2.
5. Check if `AUDIENCE_JTBD.md` exists. If it does, read it.

## Phase 2: Define Requirements

This phase produces the specs that drive everything Ralph does. Specs are the source of truth for what should be built. Both the planning and building prompts consume specs extensively.

### 2a. Gather Project Context

Ask the user these questions using the AskUserQuestion tool:

**Question 1:** What is the ultimate goal for this project? (This becomes the project-specific goal in the planning prompt.)
- Use `$ARGUMENTS` if the user already provided a goal description.

**Question 2:** Where is the application source code?
- Offer detected directories as options (e.g., `src/`, `app/`, `lib/`, `packages/`).

**Question 3:** What are the validation commands?
- Detect from package.json scripts, Makefile, Cargo.toml, etc.
- Confirm: test command, typecheck command, lint command.

### 2b. Identify Jobs to Be Done (JTBD)

A JTBD is a high-level user need or outcome the system should accomplish. Work with the user to identify the JTBDs for this project.

Use AskUserQuestion to interview the user:
- Who is the audience? What outcomes do they want?
- What are the high-level jobs the system needs to fulfill?
- Are there multiple connected audiences (e.g., "designer" creates, "client" reviews)?

Record the audience and JTBDs. If the project would benefit from audience-centric planning, create `AUDIENCE_JTBD.md` capturing who the audience is and what outcomes they want.

### 2c. Decompose into Topics of Concern

Break each JTBD into **topics of concern** — distinct, focused aspects of the system. Each topic becomes one spec file.

**Cardinality:**
- 1 JTBD → multiple topics of concern
- 1 topic of concern → 1 spec file
- 1 spec → multiple tasks (specs are larger units than tasks)

**Example:**
- JTBD: "Help designers create mood boards"
- Topics: image collection, color extraction, layout, sharing
- Each topic → one spec file (`specs/image-collection.md`, `specs/color-extraction.md`, etc.)

**Scope Test — "One Sentence Without 'And'":**

Can you describe the topic in one sentence without conjoining unrelated capabilities?
- ✓ "The color extraction system analyzes images to identify dominant colors"
- ✗ "The user system handles authentication, profiles, and billing" → this is 3 separate topics

If you need "and" to describe what it does, split it into multiple topics.

Use AskUserQuestion to walk the user through this decomposition, clarifying edge cases, constraints, and acceptance criteria for each topic.

### 2d. Write Spec Files

Create `specs/FILENAME.md` for each topic of concern. One file per topic. Use kebab-case for filenames (e.g., `specs/color-extraction.md`).

**There is no rigid template.** Let the content and structure be whatever makes most sense for the topic. However, a well-formed spec typically addresses:

1. **Purpose** — What user outcome does this topic address? Why does it matter?
2. **Functional Requirements** — What capabilities are needed? What should the system do?
3. **Acceptance Criteria** — Observable, verifiable outcomes that indicate success. These are critical for driving backpressure during building.
   - ✓ "Extracts 5-10 dominant colors from any uploaded image" (behavioral outcome)
   - ✓ "Processes images <5MB in <100ms" (measurable)
   - ✓ "Handles edge cases: grayscale, single-color, transparent backgrounds" (observable)
   - ✗ "Use K-means clustering with 3 iterations" (too prescriptive — specify WHAT to verify, not HOW to implement)
4. **Edge Cases & Constraints** — Known limitations, performance requirements, data format specs.
5. **Dependencies** — Relationships to other specs or systems.

**Key principles for specs:**
- **Behavioral, not prescriptive:** Specify WHAT success looks like, not HOW to build it.
- **Observable and measurable:** Acceptance criteria should be verifiable through tests.
- **Focused scope:** One topic per file. No "and" conjunctions joining unrelated capabilities.
- **Clear and concise:** Minimize ambiguity — Ralph reads these extensively (250-500 subagents study specs each iteration).
- **Token-efficient:** Specs load every iteration (~5,000 tokens allocated). Keep them lean and well-organized.
- **Prefer markdown over JSON** for better token efficiency.

Ask the user if they want to review/edit each spec before finalizing, or if they want you to draft them all based on the discussion.

## Phase 3: Scaffold Ralph Files

Create the following files, customized for this project. Do NOT overwrite existing files without confirmation.

### 3a. `PROMPT_plan.md`

```
0a. Study `SPECS_DIR/*` with up to 250 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md (if present) to understand the plan so far.
0c. Study `SRC_LIB_DIR/*` with up to 250 parallel Sonnet subagents to understand shared utilities & components.
0d. For reference, the application source code is in `SRC_DIR/*`.

1. Study @IMPLEMENTATION_PLAN.md (if present; it may be incorrect) and use up to 500 Sonnet subagents to study existing source code in `SRC_DIR/*` and compare it against `SPECS_DIR/*`. Use an Opus subagent to analyze findings, prioritize tasks, and create/update @IMPLEMENTATION_PLAN.md as a bullet point list sorted in priority of items yet to be implemented. Ultrathink. Consider searching for TODO, minimal implementations, placeholders, skipped/flaky tests, and inconsistent patterns. Study @IMPLEMENTATION_PLAN.md to determine starting point for research and keep it up to date with items considered complete/incomplete using subagents. For each task in the plan, derive required tests from acceptance criteria in specs — what specific outcomes need verification (behavior, performance, edge cases). Tests verify WHAT works, not HOW it's implemented. Include as part of task definition.

IMPORTANT: Plan only. Do NOT implement anything. Do NOT assume functionality is missing; confirm with code search first. Treat `SRC_LIB_DIR` as the project's standard library for shared utilities and components. Prefer consolidated, idiomatic implementations there over ad-hoc copies. When deriving test requirements from acceptance criteria, identify whether verification requires programmatic validation (measurable, inspectable) or human-like judgment (perceptual quality, tone, aesthetics). Both types are equally valid backpressure mechanisms. For subjective criteria that resist programmatic validation, explore SRC_LIB_DIR for non-deterministic evaluation patterns.

ULTIMATE GOAL: We want to achieve PROJECT_GOAL. Consider missing elements and plan accordingly. If an element is missing, search first to confirm it doesn't exist, then if needed author the specification at SPECS_DIR/FILENAME.md. If you create a new element then document the plan to implement it in @IMPLEMENTATION_PLAN.md using a subagent.
```

Replace `SPECS_DIR`, `SRC_DIR`, `SRC_LIB_DIR`, and `PROJECT_GOAL` with the actual values gathered in Phase 2.

### 3b. `PROMPT_build.md`

```
0a. Study `SPECS_DIR/*` with up to 500 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md.
0c. For reference, the application source code is in `SRC_DIR/*`.

1. Your task is to implement functionality per the specifications using parallel subagents. Follow @IMPLEMENTATION_PLAN.md and choose the most important item to address. Tasks include required tests — implement tests as part of task scope. Before making changes, search the codebase (don't assume not implemented) using Sonnet subagents. You may use up to 500 parallel Sonnet subagents for searches/reads and only 1 Sonnet subagent for build/tests. Use Opus subagents when complex reasoning is needed (debugging, architectural decisions).
2. After implementing functionality or resolving problems, run all required tests specified in the task definition. All required tests must exist and pass before the task is considered complete. If functionality is missing then it's your job to add it as per the application specifications. Ultrathink.
3. When you discover issues, immediately update @IMPLEMENTATION_PLAN.md with your findings using a subagent. When resolved, update and remove the item.
4. When the tests pass, update @IMPLEMENTATION_PLAN.md, then `git add -A` then `git commit` with a message describing the changes. After the commit, `git push`.

999. Required tests derived from acceptance criteria must exist and pass before committing. Tests are part of implementation scope, not optional. Test-driven development approach: tests can be written first or alongside implementation.
9999. Create tests to verify implementation meets acceptance criteria and include both conventional tests (behavior, performance, correctness) and perceptual quality tests (for subjective criteria, see SRC_LIB_DIR patterns).
99999. Important: When authoring documentation, capture the why — tests and implementation importance.
999999. Important: Single sources of truth, no migrations/adapters. If tests unrelated to your work fail, resolve them as part of the increment.
9999999. As soon as there are no build or test errors create a git tag. If there are no git tags start at 0.0.0 and increment patch by 1 for example 0.0.1 if 0.0.0 does not exist.
99999999. You may add extra logging if required to debug issues.
999999999. Keep @IMPLEMENTATION_PLAN.md current with learnings using a subagent — future work depends on this to avoid duplicating efforts. Update especially after finishing your turn.
9999999999. When you learn something new about how to run the application, update @AGENTS.md using a subagent but keep it brief. For example if you run commands multiple times before learning the correct command then that file should be updated.
99999999999. For any bugs you notice, resolve them or document them in @IMPLEMENTATION_PLAN.md using a subagent even if it is unrelated to the current piece of work.
999999999999. Implement functionality completely. Placeholders and stubs waste efforts and time redoing the same work.
9999999999999. When @IMPLEMENTATION_PLAN.md becomes large periodically clean out the items that are completed from the file using a subagent.
99999999999999. If you find inconsistencies in the SPECS_DIR/* then use an Opus 4.5 subagent with 'ultrathink' requested to update the specs.
999999999999999. IMPORTANT: Keep @AGENTS.md operational only — status updates and progress notes belong in `IMPLEMENTATION_PLAN.md`. A bloated AGENTS.md pollutes every future loop's context.
```

Replace `SPECS_DIR`, `SRC_DIR`, and `SRC_LIB_DIR` with actual values.

### 3c. `PROMPT_plan_work.md`

This is a variant of `PROMPT_plan.md` for scoped planning on work branches. The `${WORK_SCOPE}` variable is substituted by `loop.sh` at runtime via `envsubst`.

```
0a. Study `SPECS_DIR/*` with up to 250 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md (if present) to understand the plan so far.
0c. Study `SRC_LIB_DIR/*` with up to 250 parallel Sonnet subagents to understand shared utilities & components.
0d. For reference, the application source code is in `SRC_DIR/*`.

1. You are creating a SCOPED implementation plan for work: "${WORK_SCOPE}". Study @IMPLEMENTATION_PLAN.md (if present; it may be incorrect) and use up to 500 Sonnet subagents to study existing source code in `SRC_DIR/*` and compare it against `SPECS_DIR/*`. Use an Opus subagent to analyze findings, prioritize tasks, and create/update @IMPLEMENTATION_PLAN.md as a bullet point list sorted in priority of items yet to be implemented. Ultrathink. Consider searching for TODO, minimal implementations, placeholders, skipped/flaky tests, and inconsistent patterns. Study @IMPLEMENTATION_PLAN.md to determine starting point for research and keep it up to date with items considered complete/incomplete using subagents.

IMPORTANT: This is SCOPED PLANNING for "${WORK_SCOPE}" only. Create a plan containing ONLY tasks directly related to this work scope. Be conservative — if uncertain whether a task belongs to this work, exclude it. The plan can be regenerated if too narrow. Plan only. Do NOT implement anything. Do NOT assume functionality is missing; confirm with code search first. Treat `SRC_LIB_DIR` as the project's standard library for shared utilities and components. Prefer consolidated, idiomatic implementations there over ad-hoc copies.

ULTIMATE GOAL: We want to achieve the scoped work "${WORK_SCOPE}". Consider missing elements related to this work and plan accordingly. If an element is missing, search first to confirm it doesn't exist, then if needed author the specification at SPECS_DIR/FILENAME.md. If you create a new element then document the plan to implement it in @IMPLEMENTATION_PLAN.md using a subagent.
```

Replace `SPECS_DIR`, `SRC_DIR`, and `SRC_LIB_DIR` with actual values.

### 3d. `AGENTS.md`

```
## Build & Run

BUILD_INSTRUCTIONS

## Validation

Run these after implementing to get immediate feedback:

- Tests: `TEST_COMMAND`
- Typecheck: `TYPECHECK_COMMAND`
- Lint: `LINT_COMMAND`

## Operational Notes

OPERATIONAL_NOTES

### Codebase Patterns

CODEBASE_PATTERNS
```

Replace placeholders with detected values. For unknown commands, leave as `[not configured]` and note it.

### 3e. `IMPLEMENTATION_PLAN.md`

```
<!-- Generated by LLM with content and structure it deems most appropriate -->
```

Only create this if it does not already exist.

### 3f. `loop.sh`

```bash
#!/bin/bash
set -euo pipefail

# Usage:
#   ./loop.sh [plan] [max_iterations]       # Plan/build on current branch
#   ./loop.sh plan-work "work description"  # Create scoped plan on work branch
# Examples:
#   ./loop.sh              # Build mode, unlimited iterations
#   ./loop.sh 20           # Build mode, max 20 iterations
#   ./loop.sh plan         # Plan mode, unlimited iterations
#   ./loop.sh plan 5       # Plan mode, max 5 iterations
#   ./loop.sh plan-work "user authentication with OAuth"

# Parse arguments
MODE="build"
PROMPT_FILE="PROMPT_build.md"
MAX_ITERATIONS=0

if [ "${1:-}" = "plan" ]; then
    MODE="plan"
    PROMPT_FILE="PROMPT_plan.md"
    MAX_ITERATIONS=${2:-0}
elif [ "${1:-}" = "plan-work" ]; then
    if [ -z "${2:-}" ]; then
        echo "Error: plan-work requires a work description"
        echo "Usage: ./loop.sh plan-work \"description of the work\""
        exit 1
    fi
    MODE="plan-work"
    WORK_DESCRIPTION="$2"
    PROMPT_FILE="PROMPT_plan_work.md"
    MAX_ITERATIONS=${3:-5}
elif [[ "${1:-}" =~ ^[0-9]+$ ]]; then
    MAX_ITERATIONS=$1
fi

ITERATION=0
CURRENT_BRANCH=$(git branch --show-current)

# Validate branch for plan-work mode
if [ "$MODE" = "plan-work" ]; then
    if [ "$CURRENT_BRANCH" = "main" ] || [ "$CURRENT_BRANCH" = "master" ]; then
        echo "Error: plan-work should be run on a work branch, not main/master"
        echo "Create a work branch first: git checkout -b ralph/your-work"
        exit 1
    fi
    export WORK_SCOPE="$WORK_DESCRIPTION"
fi

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "Mode:   $MODE"
echo "Prompt: $PROMPT_FILE"
echo "Branch: $CURRENT_BRANCH"
[ "$MODE" = "plan-work" ] && echo "Work:   $WORK_DESCRIPTION"
[ $MAX_ITERATIONS -gt 0 ] && echo "Max:    $MAX_ITERATIONS iterations"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ ! -f "$PROMPT_FILE" ]; then
    echo "Error: $PROMPT_FILE not found"
    exit 1
fi

while true; do
    if [ $MAX_ITERATIONS -gt 0 ] && [ $ITERATION -ge $MAX_ITERATIONS ]; then
        echo "Reached max iterations: $MAX_ITERATIONS"
        [ "$MODE" = "plan-work" ] && echo "Scoped plan created. To build, run: ./loop.sh 20"
        break
    fi

    if [ "$MODE" = "plan-work" ]; then
        envsubst < "$PROMPT_FILE" | claude -p \
            --dangerously-skip-permissions \
            --output-format=stream-json \
            --model opus \
            --verbose
    else
        cat "$PROMPT_FILE" | claude -p \
            --dangerously-skip-permissions \
            --output-format=stream-json \
            --model opus \
            --verbose
    fi

    git push origin "$CURRENT_BRANCH" || {
        echo "Failed to push. Creating remote branch..."
        git push -u origin "$CURRENT_BRANCH"
    }

    ITERATION=$((ITERATION + 1))
    echo -e "\n\n======================== LOOP $ITERATION ========================\n"
done
```

### 3g. `specs/` directory

If it does not exist, create `specs/` and populate it with the spec files authored in Phase 2d.

### 3h. `AUDIENCE_JTBD.md` (optional)

If the project has a clear audience with multiple JTBDs and would benefit from audience-centric planning (e.g., product-facing applications), create `AUDIENCE_JTBD.md` capturing:
- Who the audience is
- What outcomes (JTBDs) they want
- How JTBDs relate to the topics of concern in `specs/*`

This enables SLC-oriented planning where the planning prompt can reason about which activities at what capability depths form the most valuable next release.

## Phase 4: Run Planning Analysis

After scaffolding is complete, ask the user if they want to run the planning analysis now (equivalent to `./loop.sh plan 1` but within this session):

1. Study `specs/*` to understand requirements.
2. Study the source code with parallel subagents.
3. Compare specs against existing code — find gaps, TODOs, placeholders, missing tests, inconsistent patterns.
4. For each identified task, derive required tests from acceptance criteria in the relevant spec.
5. Create/update `IMPLEMENTATION_PLAN.md` with a prioritized bullet list of work items, each including test requirements.
6. Do NOT implement anything — planning only.

## Phase 5: Summary

After planning, output a summary:

```
Ralph Loop Setup Complete
━━━━━━━━━━━━━━━━━━━━━━━━

Files created:
  - PROMPT_plan.md (planning mode prompt)
  - PROMPT_build.md (building mode prompt)
  - PROMPT_plan_work.md (scoped planning for work branches)
  - AGENTS.md (operational guide)
  - IMPLEMENTATION_PLAN.md (task backlog)
  - loop.sh (orchestration script)
  - specs/ (specifications directory)

Specs created:
  - specs/FILENAME.md (list each spec)

To run Ralph:
  chmod +x loop.sh
  ./loop.sh plan 5     # Plan for up to 5 iterations
  ./loop.sh 20         # Build for up to 20 iterations

Work branches:
  git checkout -b ralph/feature-name
  ./loop.sh plan-work "description of the work"
  ./loop.sh 20         # Build from scoped plan

Key concepts:
  - IMPLEMENTATION_PLAN.md is shared state between loop iterations
  - Each iteration gets fresh context, reads the plan, picks a task
  - AGENTS.md must stay lean (~60 lines) — it loads every iteration
  - specs/* define what to build; the plan tracks progress
  - loop.sh plan = gap analysis only, no implementation
  - loop.sh = build mode, implements + tests + commits
  - loop.sh plan-work = scoped planning for a work branch
```

## Important Principles

### Context Is Everything

Claude has ~176K usable tokens per iteration, with 40-60% utilization being the "smart zone." This drives everything:
- **One task per loop iteration.** Tight scope = 100% smart zone utilization.
- **Use subagents as memory extension.** Each subagent gets ~156K that's garbage collected. Fan out to avoid polluting main context.
- **Keep files lean.** AGENTS.md ~60 lines. Specs concise. ~5,000 tokens allocated for specs at start of each iteration.
- **Prefer markdown over JSON** for token efficiency.

### Steering Ralph

You steer from two directions:

- **Upstream (patterns):** Specs and existing code patterns guide what Ralph builds. If Ralph generates wrong patterns, add/update utilities and code patterns to steer it toward correct ones. Every loop's context starts from a known state (`PROMPT.md` + `AGENTS.md`).
- **Downstream (backpressure):** Tests, typechecks, lints, and builds reject invalid work. The prompt says "run tests" generically; `AGENTS.md` specifies actual commands to make backpressure project-specific. Acceptance criteria in specs drive test requirements — tasks can't be marked done without required tests passing.

### Specs Are Source of Truth

- One file per topic of concern. Focused scope, no "and" conjunctions.
- Acceptance criteria are behavioral outcomes, not implementation prescriptions.
- Specs are consumed extensively by both planning and building modes (250-500 subagents).
- Rarely updated — but can be refined if code analysis reveals inconsistencies (use Opus subagent with ultrathink).
- Specs → plan → code is the core relationship: specs define requirements, the plan tracks progress, building delivers to spec.

### Let Ralph Ralph

- Lean into the LLM's ability to self-identify, self-correct, and self-improve.
- Applies to implementation plan, task definition, and prioritization.
- Eventual consistency achieved through iteration.
- The plan is disposable — if IMPLEMENTATION_PLAN.md is wrong, regenerate it with `./loop.sh plan 1`.

### Move Outside the Loop

Your job is to sit ON the loop, not IN it. Engineer the setup and environment that allows Ralph to succeed:
- **Observe and course correct** — especially early on, watch what patterns emerge and where Ralph goes wrong.
- **Tune it like a guitar** — instead of prescribing everything upfront, observe and adjust reactively. When Ralph fails a specific way, add a sign to help him next time.
- Signs aren't just prompt text — they're anything Ralph can discover: prompt guardrails, AGENTS.md learnings, utilities in the codebase, specs.

### Fan Out Reads, Serialize Writes

Use many parallel subagents for searching/reading, only 1 for building/testing. This maximizes context utilization without race conditions.

### Don't Assume Not Implemented

Always search the codebase first before planning new work. This is Ralph's Achilles' heel — the prompt explicitly says "don't assume not implemented" because without it, Ralph will plan work that already exists.

### Work Branches

For feature-scoped work, scope at plan creation (deterministic), not at task selection (probabilistic):
1. Create a work branch: `git checkout -b ralph/feature-name`
2. Run scoped planning: `./loop.sh plan-work "description of the work"`
3. Build from scoped plan: `./loop.sh 20`
4. PR when complete: `gh pr create --base main`

The scoped plan contains ONLY tasks related to the work description. Ralph picks "most important" from an already-scoped plan — no runtime filtering needed.
