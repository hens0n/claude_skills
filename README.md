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

### usaspending-search

Searches and analyzes U.S. federal government spending data using the [USAspending.gov](https://usaspending.gov) API. Find contracts, grants, loans, and other federal awards by keyword, agency, recipient, location, and time period.

**Usage:**

```
/usaspending-search [search query or question about federal spending]
```

**What it does:**

1. Parses natural language questions about federal spending
2. Builds API requests with appropriate filters (keywords, date ranges, agencies, recipients, locations, NAICS/PSC codes)
3. Calls the USAspending.gov API endpoints
4. Presents results in formatted tables and summaries with human-readable dollar amounts

**Supported queries:**

| Query Type | Example |
|---|---|
| Spending totals | "How much has been spent on cybersecurity?" |
| Top recipients | "Who are the top contractors for the Department of Defense?" |
| Company awards | "What contracts has Lockheed Martin received?" |
| Geographic spending | "What federal money goes to California?" |
| Spending trends | "Show me NASA spending trends over the last 5 years" |
| Award details | "Tell me about award ID W911NF-20-1-0001" |

**API endpoints used:** Award search, spending over time, spending by category (agency, recipient, NAICS, geography), award counts, individual award details, agency info, and recipient profiles.

---

### sam-gov-search

Searches [SAM.gov](https://sam.gov) (System for Award Management) for federal contract opportunities, assistance listings, entity information, and wage determinations using a browser. Use this to find government contracts, RFPs, RFIs, solicitations, and other federal procurement opportunities.

**Usage:**

```
/sam-gov-search [search keywords] [optional: domain] [optional: notice type]
```

**What it does:**

1. Opens SAM.gov in a browser via Playwright
2. Selects the appropriate search domain and enters keywords
3. Applies filters (notice type, status, etc.)
4. Extracts and presents structured results with titles, notice IDs, agencies, due dates, and descriptions
5. Can drill into detail pages for full descriptions, contact info, and attachments

**Supported domains:**

| Request | Domain |
|---|---|
| Contracts, RFPs, RFIs, solicitations (default) | Contract Opportunities |
| Grants, assistance, CFDA | Assistance Listings |
| Entities, vendors, companies | Entity Information |
| Agencies, organizations | Federal Hierarchy |
| Wages, labor rates | Wage Determinations |

**Notice type filters** (Contract Opportunities): Special Notice, Sources Sought, Presolicitation, Solicitation, Combined Synopsis/Solicitation, Award Notice, Justification, Sale of Surplus Property.

---

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