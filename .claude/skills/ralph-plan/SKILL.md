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
4. Check if a `specs/` directory exists. If not, note this — we will need to create it.

## Phase 2: Gather Requirements

Ask the user these questions using the AskUserQuestion tool:

**Question 1:** What is the ultimate goal for this project? (This becomes the [project-specific goal] in the planning prompt.)
- Use `$ARGUMENTS` if the user already provided a goal description.

**Question 2:** Where is the application source code?
- Offer detected directories as options (e.g., `src/`, `app/`, `lib/`, `packages/`).

**Question 3:** Does the project have existing specs or requirements documentation?
- If `specs/` exists, confirm it.
- If not, ask if the user wants to create a `specs/` directory and seed it with initial spec files.

**Question 4:** What are the validation commands?
- Detect from package.json scripts, Makefile, Cargo.toml, etc.
- Confirm: test command, typecheck command, lint command.

## Phase 3: Scaffold Ralph Files

Create the following files, customized for this project. Do NOT overwrite existing files without confirmation.

### 3a. `PROMPT_plan.md`

```
0a. Study `SPECS_DIR/*` with up to 250 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md (if present) to understand the plan so far.
0c. Study `SRC_LIB_DIR/*` with up to 250 parallel Sonnet subagents to understand shared utilities & components.
0d. For reference, the application source code is in `SRC_DIR/*`.

1. Study @IMPLEMENTATION_PLAN.md (if present; it may be incorrect) and use up to 500 Sonnet subagents to study existing source code in `SRC_DIR/*` and compare it against `SPECS_DIR/*`. Use an Opus subagent to analyze findings, prioritize tasks, and create/update @IMPLEMENTATION_PLAN.md as a bullet point list sorted in priority of items yet to be implemented. Ultrathink. Consider searching for TODO, minimal implementations, placeholders, skipped/flaky tests, and inconsistent patterns. Study @IMPLEMENTATION_PLAN.md to determine starting point for research and keep it up to date with items considered complete/incomplete using subagents.

IMPORTANT: Plan only. Do NOT implement anything. Do NOT assume functionality is missing; confirm with code search first. Treat `SRC_LIB_DIR` as the project's standard library for shared utilities and components. Prefer consolidated, idiomatic implementations there over ad-hoc copies.

ULTIMATE GOAL: We want to achieve PROJECT_GOAL. Consider missing elements and plan accordingly. If an element is missing, search first to confirm it doesn't exist, then if needed author the specification at SPECS_DIR/FILENAME.md. If you create a new element then document the plan to implement it in @IMPLEMENTATION_PLAN.md using a subagent.
```

Replace `SPECS_DIR`, `SRC_DIR`, `SRC_LIB_DIR`, and `PROJECT_GOAL` with the actual values gathered in Phase 2.

### 3b. `PROMPT_build.md`

```
0a. Study `SPECS_DIR/*` with up to 500 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md.
0c. For reference, the application source code is in `SRC_DIR/*`.

1. Your task is to implement functionality per the specifications using parallel subagents. Follow @IMPLEMENTATION_PLAN.md and choose the most important item to address. Before making changes, search the codebase (don't assume not implemented) using Sonnet subagents. You may use up to 500 parallel Sonnet subagents for searches/reads and only 1 Sonnet subagent for build/tests. Use Opus subagents when complex reasoning is needed (debugging, architectural decisions).
2. After implementing functionality or resolving problems, run the tests for that unit of code that was improved. If functionality is missing then it's your job to add it as per the application specifications. Ultrathink.
3. When you discover issues, immediately update @IMPLEMENTATION_PLAN.md with your findings using a subagent. When resolved, update and remove the item.
4. When the tests pass, update @IMPLEMENTATION_PLAN.md, then `git add -A` then `git commit` with a message describing the changes. After the commit, `git push`.

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

Replace `SPECS_DIR` and `SRC_DIR` with actual values.

### 3c. `AGENTS.md`

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

### 3d. `IMPLEMENTATION_PLAN.md`

```
<!-- Generated by LLM with content and structure it deems most appropriate -->
```

Only create this if it does not already exist.

### 3e. `loop.sh`

```bash
#!/bin/bash
# Usage: ./loop.sh [plan] [max_iterations]
# Examples:
#   ./loop.sh              # Build mode, unlimited iterations
#   ./loop.sh 20           # Build mode, max 20 iterations
#   ./loop.sh plan         # Plan mode, unlimited iterations
#   ./loop.sh plan 5       # Plan mode, max 5 iterations

# Parse arguments
if [ "$1" = "plan" ]; then
    MODE="plan"
    PROMPT_FILE="PROMPT_plan.md"
    MAX_ITERATIONS=${2:-0}
elif [[ "$1" =~ ^[0-9]+$ ]]; then
    MODE="build"
    PROMPT_FILE="PROMPT_build.md"
    MAX_ITERATIONS=$1
else
    MODE="build"
    PROMPT_FILE="PROMPT_build.md"
    MAX_ITERATIONS=0
fi

ITERATION=0
CURRENT_BRANCH=$(git branch --show-current)

echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
echo "Mode:   $MODE"
echo "Prompt: $PROMPT_FILE"
echo "Branch: $CURRENT_BRANCH"
[ $MAX_ITERATIONS -gt 0 ] && echo "Max:    $MAX_ITERATIONS iterations"
echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"

if [ ! -f "$PROMPT_FILE" ]; then
    echo "Error: $PROMPT_FILE not found"
    exit 1
fi

while true; do
    if [ $MAX_ITERATIONS -gt 0 ] && [ $ITERATION -ge $MAX_ITERATIONS ]; then
        echo "Reached max iterations: $MAX_ITERATIONS"
        break
    fi

    cat "$PROMPT_FILE" | claude -p \
        --dangerously-skip-permissions \
        --output-format=stream-json \
        --model opus \
        --verbose

    git push origin "$CURRENT_BRANCH" || {
        echo "Failed to push. Creating remote branch..."
        git push -u origin "$CURRENT_BRANCH"
    }

    ITERATION=$((ITERATION + 1))
    echo -e "\n\n======================== LOOP $ITERATION ========================\n"
done
```

### 3f. `specs/` directory

If it does not exist, create `specs/` and optionally seed it with an initial spec file based on the project goal. Ask the user if they want you to draft initial specs.

## Phase 4: Run Planning Analysis

After scaffolding is complete, ask the user if they want to run the planning analysis now (equivalent to `./loop.sh plan 1` but within this session):

1. Study `specs/*` to understand requirements.
2. Study the source code with parallel subagents.
3. Compare specs against existing code — find gaps, TODOs, placeholders, missing tests, inconsistent patterns.
4. Create/update `IMPLEMENTATION_PLAN.md` with a prioritized bullet list of work items.
5. Do NOT implement anything — planning only.

## Phase 5: Summary

After planning, output a summary:

```
Ralph Loop Setup Complete
━━━━━━━━━━━━━━━━━━━━━━━━

Files created:
  - PROMPT_plan.md (planning mode prompt)
  - PROMPT_build.md (building mode prompt)
  - AGENTS.md (operational guide)
  - IMPLEMENTATION_PLAN.md (task backlog)
  - loop.sh (orchestration script)
  - specs/ (specifications directory)

To run Ralph:
  chmod +x loop.sh
  ./loop.sh plan 5     # Plan for up to 5 iterations
  ./loop.sh 20         # Build for up to 20 iterations

Key concepts:
  - IMPLEMENTATION_PLAN.md is shared state between loop iterations
  - Each iteration gets fresh context, reads the plan, picks a task
  - AGENTS.md must stay lean (~60 lines) — it loads every iteration
  - specs/* define what to build; the plan tracks progress
  - loop.sh plan = gap analysis only, no implementation
  - loop.sh = build mode, implements + tests + commits
```

## Important Principles

- **Context is everything.** Claude has ~176K usable tokens per iteration. Keep files lean. One task per loop iteration.
- **Upstream steering:** specs and patterns guide what Ralph builds.
- **Downstream steering:** tests and backpressure prevent placeholder implementations.
- **The plan is disposable.** If IMPLEMENTATION_PLAN.md is wrong, regenerate it with `./loop.sh plan 1`.
- **Fan out reads, serialize writes.** Use many parallel subagents for searching/reading, only 1 for building/testing.
- **Don't assume not implemented.** Always search the codebase first before planning new work.
- **AGENTS.md stays operational only.** No status updates, no progress notes — those go in IMPLEMENTATION_PLAN.md.
