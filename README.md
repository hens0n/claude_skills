# claude_skills

A collection of custom [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for automating development workflows.

## Skills

### ralph-plan

Scaffolds and plans a **Ralph Wiggum autonomous loop** — an iterative AI development methodology where Claude is invoked repeatedly in a bash loop. Each iteration gets a fresh context window, reads a persistent implementation plan from disk, picks the highest-priority task, implements it, commits, and exits. The loop then restarts.

**Usage:**

```
/ralph-plan [goal-description]
```

**What it creates:**

| File | Purpose |
|---|---|
| `PROMPT_plan.md` | Planning mode prompt — gap analysis between specs and code |
| `PROMPT_build.md` | Build mode prompt — implements, tests, and commits |
| `AGENTS.md` | Operational guide (build commands, validation, codebase patterns) |
| `IMPLEMENTATION_PLAN.md` | Prioritized task backlog shared across iterations |
| `loop.sh` | Bash orchestration script |
| `specs/` | Specifications directory |

**Running the loop:**

```bash
chmod +x loop.sh
./loop.sh plan 5     # Plan for up to 5 iterations
./loop.sh 20         # Build for up to 20 iterations
./loop.sh            # Build mode, unlimited
```

**Key concepts:**

- Each iteration gets fresh context (~176K tokens) and reads `IMPLEMENTATION_PLAN.md` from disk
- Fan out reads with parallel subagents, serialize writes
- Specs steer what gets built; tests prevent placeholder implementations
- `AGENTS.md` must stay lean (~60 lines) since it loads every iteration

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