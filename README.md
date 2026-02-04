# claude_skills

A collection of custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for automating development workflows.

## Skills

### ralph-plan

Scaffolds and plans a **Ralph Wiggum autonomous loop** — an iterative AI development methodology where Claude is invoked repeatedly in a bash loop. Each iteration gets a fresh context window, reads a persistent implementation plan from disk, picks the highest-priority task, implements it, commits, and exits. The loop then restarts.

**Usage:**

```
/ralph-plan [goal-description]
```

**What it does:**

1. **Studies the project** — detects languages, frameworks, build tools, existing patterns
2. **Defines requirements** — interviews you to identify Jobs to Be Done (JTBD), decomposes them into topics of concern, and writes focused spec files with acceptance criteria
3. **Scaffolds Ralph files** — creates all the prompts, operational guide, and loop script
4. **Runs initial planning** — gap analysis between specs and code to populate the implementation plan

**What it creates:**

| File | Purpose |
|---|---|
| `specs/*.md` | One spec per topic of concern — source of truth for what to build |
| `PROMPT_plan.md` | Planning mode prompt — gap analysis between specs and code |
| `PROMPT_build.md` | Build mode prompt — implements, tests, and commits |
| `PROMPT_plan_work.md` | Scoped planning for work branches |
| `AGENTS.md` | Operational guide (build commands, validation, codebase patterns) |
| `IMPLEMENTATION_PLAN.md` | Prioritized task backlog shared across iterations |
| `loop.sh` | Bash orchestration script (plan, build, and plan-work modes) |
| `AUDIENCE_JTBD.md` | *(optional)* Audience and JTBD definitions for SLC-oriented planning |

**Running the loop:**

```bash
chmod +x loop.sh
./loop.sh plan 5     # Plan for up to 5 iterations
./loop.sh 20         # Build for up to 20 iterations
./loop.sh            # Build mode, unlimited
```

**Work branches** (scoped planning):

```bash
git checkout -b ralph/feature-name
./loop.sh plan-work "user authentication with OAuth"
./loop.sh 20         # Build from scoped plan
```

**Spec methodology:**

Specs follow the JTBD decomposition pattern:
- 1 JTBD → multiple **topics of concern** → 1 spec file each → multiple tasks
- Each topic passes the "one sentence without 'and'" scope test
- Acceptance criteria are behavioral outcomes, not implementation prescriptions
- Specs drive **acceptance-driven backpressure** — tasks include required tests derived from acceptance criteria

**Key concepts:**

- Each iteration gets fresh context (~176K tokens) and reads `IMPLEMENTATION_PLAN.md` from disk
- Fan out reads with parallel subagents, serialize writes
- **Upstream steering:** specs and code patterns guide what Ralph builds
- **Downstream steering:** tests and backpressure prevent placeholder implementations
- `AGENTS.md` must stay lean (~60 lines) since it loads every iteration
- The plan is disposable — regenerate with `./loop.sh plan 1` if it goes wrong

## Adding Skills

Skills live in `.claude/skills/` as directories containing a `SKILL.md` file. The SKILL.md frontmatter defines the skill name, description, allowed tools, and argument hints. See the [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code/skills) for the full specification.

```
.claude/
└── skills/
    └── your-skill/
        └── SKILL.md
```

## License

MIT