# LGTM

An orchestration framework that turns AI coding agents into a managed engineering team.

You define the work as plans. LGTM breaks them into isolated issues, dispatches specialized agents in parallel, and routes every change through a PR you review and merge. Nothing ships without your sign-off.

Currently built for Claude Code. Designed to work with any agentic coding tool.

## The problem

AI coding agents are powerful, but they have no memory of what they're working on, no structure for managing multiple tasks, and no way to coordinate parallel work without stepping on each other. You end up babysitting each session, manually tracking what's done, and hoping nothing gets committed that shouldn't.

LGTM gives your agents a project management layer. Plans become issues. Issues become branches. Branches become PRs. Status stays current. Agents work in isolation. You stay in control.

## How it works

Work flows through a structured hierarchy:

```
Initiative (the repo)
  └── Project        plan file + base branch + GitHub milestone
        └── Milestone    section in the plan
              └── Issue      spec file + branch + PR
```

The lifecycle is straightforward:

1. **Plan** — You describe what needs to happen. The orchestrator creates a plan with milestones and a dependency graph.
2. **Issues** — The issue writer agent reads the plan and produces self-contained spec files. Each one has acceptance criteria, test cases, file paths, and dependencies.
3. **Dispatch** — The orchestrator reads the dependency graph and dispatches all ready issues in parallel, each in its own git worktree.
4. **Execute** — Each agent gets only its issue file. No conversation history, no shared state, no assumptions about what came before.
5. **PR** — Agent pushes its branch, opens a PR targeting the project's base branch. Issue ID goes in the PR title.
6. **You merge** — You review every PR on GitHub. That's the only way code lands.
7. **Repeat** — Merged PRs unlock dependent issues. The orchestrator picks them up and dispatches the next batch.

### The issue-to-PR contract

Every issue file has an ID like `P2M1-003`. The PR title starts with that same ID: `P2M1-003: Add rate limiting middleware`. That's how the framework links a spec on disk to a pull request on GitHub. Branch names are just descriptive slugs for humans.

### Parallel execution with worktrees

Each agent working on code gets its own git worktree — a separate working directory with its own branch, forked from the project's base. Three agents can work on three issues simultaneously without touching each other's files.

```
.claude/worktrees/
  ├── P2M1-001/  →  branch: auth-oauth-setup
  ├── P2M1-002/  →  branch: auth-callback-handler
  └── P2M2-001/  →  branch: auth-rate-limiting
```

Worktrees are disposable. If one gets lost, re-dispatch the agent from the same issue file. The spec is the durable record, not the worktree.

### Cold-start agents

Agents don't read conversation history. They don't know what happened last session. The orchestrator hands each agent a self-contained prompt with everything it needs: the issue spec, file paths, constraints. This makes agents restartable, replaceable, and parallelizable.

## What ships with LGTM

### Agents

Seven agent definitions in `.claude/agents/`, each with a defined role, model, toolset, and skills:

| Agent | Does what | Model |
|-------|-----------|-------|
| **Orchestrator** | Runs the full lifecycle. Creates plans, dispatches agents, tracks status, reconciles contributions. | Opus |
| **Issue writer** | Reads plans, walks the codebase, produces spec files grounded in actual code. | Opus |
| **Fullstack engineer** | Implements code changes. Security-first, type-safe, minimal surface area. | Opus |
| **Content editor** | Writes and optimizes marketing copy, SEO, structured data. | Opus |
| **Humanizer** | Removes AI writing patterns through a draft-critique-revise loop with a mechanical grep gate. | Sonnet |
| **QA** | Tests live sites with a headless browser. Produces reports with screenshots, videos, and repro steps. | Sonnet |
| **Translator** | Translates JSON dictionaries and MDX files with structure validation and anti-fabrication checks. | Opus |

### Skills

Sixteen skill files in `.claude/skills/`, each a structured playbook that agents (or you) can invoke:

**Framework**
- `workflow` — The core playbook. Plan-to-PR lifecycle, hierarchy model, git conventions, templates.
- `onboard` — Scans for `{CONFIGURE:}` placeholders and walks through project setup interactively.
- `humanizer` — 25 documented AI writing patterns with detection rules and fix strategies.
- `dogfood` — Six-phase exploratory testing. Systematic exploration with full reproduction evidence.
- `test-strategy` — Project-specific QA checklist (template, you fill it in).
- `agent-browser` — Headless browser automation via a fast Rust client.
- `translator` — Translation rules, terminology glossary, anti-fabrication checks, locale-specific guides.

**Content and SEO**
- `copywriting` — Conversion-focused copy with frameworks and formulas.
- `copy-editing` — Seven-sweep refinement: clarity, voice, so-what, prove-it, specificity, emotion, risk.
- `content-strategy` — Topic clusters, buyer stage mapping, editorial planning.
- `page-cro` — Landing page conversion optimization.
- `seo-audit` — Technical SEO audits.
- `ai-seo` — Optimization for AI search engines (ChatGPT, Perplexity, AI Overviews).
- `schema-markup` — JSON-LD structured data generation.

**Utility**
- `tree` — Project structure visualization.
- `compare` — Side-by-side diff viewer.

### Scripts

Python utilities in `.claude/scripts/`:

| Script | What it does |
|--------|-------------|
| `now.py` | Returns the current timestamp with millisecond precision. Agents use this instead of guessing the time. |
| `tree.py` | Prints project structure, excluding noise directories. |
| `validate-json-structure.py` | Compares two JSON files for structural equality (same keys, nesting, types). |
| `validate-mdx-structure.py` | Compares two MDX files for structural equality (same frontmatter, non-text fields byte-identical). |
| `dictionary-diff.py` | Compares translated dictionaries across locales. |
| `log-entry.py` | Appends timestamped entries to the daily log. Lives in `skills/workflow/scripts/`. |

### Rules

Hard constraints in `.claude/rules/` that every agent follows:

- **agent-protocol** — Cold-start inputs, timestamp discipline, no AI attribution, stage specific files only, write to assigned paths, report back to orchestrator.
- **git-workflow** — Never `git add -A`. Never commit to protected branches. One branch per issue, one PR per issue. Push before creating PRs. No local merges.
- **artifact-storage** — All agent output goes to `.lgtm/ai/process-artifacts/{project}/{milestone}/{issue}/`. Never deleted. The persistent record of what each agent produced.

## Git model

```
master
  └── develop
        └── {project}-base
              ├── {project}-feature-a    ← one issue
              ├── {project}-feature-b    ← one issue
              └── {project}-feature-c    ← one issue
```

Issue PRs target the project's base branch. When all milestones are done, one project PR rolls the base into `develop`. You test on `develop`, then merge to `master`. Every step is a PR you control.

Commits explain **why**, not what. No conventional-commit prefixes. No `Co-Authored-By` trailers. No AI attribution. Clean history.

## Status tracking

`.lgtm/STATUS.md` is the single status dashboard — initiative health, project progress, active work, and a table for unplanned contributions that arrive outside the plan/issue pipeline.

Daily logs go to `.lgtm/ai/logs/YYYY-MM-DD/LOG.md`, appended by the `log-entry.py` script with real timestamps.

## Getting started

LGTM uses `{CONFIGURE:}` placeholder tokens in template files. These are designed to be filled in per-project.

### 1. Drop LGTM into your project

Copy the `.claude/` and `.lgtm/` directories into your project root.

### 2. Run onboard

```
/onboard
```

Or let it detect automatically — the `SessionStart` hook runs a placeholder scan and prompts you if anything is unfilled. The onboard skill walks through four categories:

- **Core** — Project name, initiative goal, build commands
- **Agents** — Brand voice (content editor), tech stack (fullstack engineer)
- **Testing** — QA checklist, build validation commands (can skip)
- **Translation** — i18n framework, locales, file paths (can skip)

Skipped categories get marked `[Not applicable]` so the scan stays clean.

### 3. Create a plan

Tell the orchestrator what you want to build. It creates a plan at `.lgtm/shared/plans/p{N}-{slug}/PLAN.md` with milestones, a dependency DAG, and a file ownership table that prevents two concurrent agents from touching the same file.

### 4. Let it run

The orchestrator dispatches the issue writer to create specs, then dispatches agents to execute them. You review and merge PRs as they come in. Status updates happen automatically.

## Project structure

LGTM lives in two top-level directories. `.claude/` holds the agent framework — definitions, skills, rules, and orchestration. `.lgtm/` holds everything that agents and humans produce while working — plans, artifacts, resources, learnings, and personal workspace.

The split is intentional. `.claude/` is the framework. `.lgtm/` is the workspace. Separating them means the framework can be updated independently of the work product, and the workspace structure works regardless of which agent framework you're using.

### `.claude/` — the framework

```
.claude/
  ├── CLAUDE.md              # Project config (template with {CONFIGURE:} tokens)
  ├── settings.json          # Hooks and plugin config
  ├── agents/                # 7 agent definitions
  │   ├── orchestrator.md
  │   ├── issue-writer.md
  │   ├── fullstack-engineer.md
  │   ├── content-editor.md
  │   ├── humanizer.md
  │   ├── qa.md
  │   └── translator.md
  ├── rules/                 # Hard constraints every agent follows
  ├── scripts/               # Timestamp, validation, tree utilities
  ├── skills/                # 16 skill playbooks with references and templates
  │   ├── workflow/          # Core: plan → issue → PR lifecycle
  │   ├── onboard/           # First-run project configuration
  │   ├── humanizer/         # AI writing pattern removal
  │   ├── dogfood/           # Exploratory QA testing
  │   ├── copywriting/       # Marketing copy frameworks
  │   ├── copy-editing/      # Seven-sweep content refinement
  │   └── ...                # + 10 more (SEO, translation, CRO, etc.)
  ├── logs/                  # Daily operational logs (created at runtime)
  └── worktrees/             # Git worktrees for parallel execution (disposable)
```

### `.lgtm/` — the workspace

Three spaces, each with a clear owner:

```
.lgtm/
  ├── STATUS.md              # Initiative health dashboard (the golden compass)
  ├── ai/                    # Agent space — agents write here, humans review
  │   ├── process-artifacts/ # All agent output, mirroring the project hierarchy
  │   └── logs/              # Agent execution logs
  ├── user/                  # Human space — your backups, configs, archives
  │   ├── backups/           # Editor settings, config snapshots
  │   └── _archive/          # Retired plans, old agent versions
  └── shared/                # Shared space — both agents and humans read/write
      ├── plans/             # All project plans and issue specs
      ├── learnings/         # Patterns discovered during work
      └── resources/         # Reference repos, datasets, templates not yet integrated into skills
```

**`ai/`** is where agents store everything they produce. Reports, rewrites, enriched data, translations. The `process-artifacts/` folder mirrors the project hierarchy (`{project}/{milestone}/{issue}/`), so every output is traceable to the issue that produced it. Artifacts are never deleted.

**`user/`** is yours. Backups, editor configs, archived plans, scratch files. Agents don't write here.

**`shared/`** is the handoff zone. `plans/` is where all project plans and issue specs live — the structured hierarchy (`p{N}-{slug}/PLAN.md`, issues, milestones) plus any informal planning. `learnings/` captures patterns that both you and agents benefit from. `resources/` holds reference material — full repos, datasets, or templates that haven't been formally integrated into skills yet.

## Design decisions

**Why filesystem specs instead of GitHub Issues?**
Agent-executable specs need structured frontmatter, dependency fields, test cases, and file path listings in a machine-readable format. GitHub Issues are conversation threads. Our issue files are cold-start contracts.

**Why one PR per issue?**
Fault isolation. A bad agent output is just a rejected PR. It doesn't block anything except issues that explicitly depend on it. No cascading failures, no rollback nightmares.

**Why no AI attribution in commits?**
`Co-Authored-By` trailers for AI accounts show as "Unverified" on GitHub and add noise to the history. The framework strips them by design.

**Why worktrees instead of just branches?**
Branches share a working directory. If two agents run simultaneously on different branches, they'll overwrite each other's uncommitted changes. Worktrees give each agent its own directory. True isolation.

**Why `now.py` for timestamps?**
LLMs can't tell time. They hallucinate dates confidently. Every timestamp in LGTM comes from a real clock via `python .claude/scripts/now.py`.

## Roadmap

LGTM ships with Claude Code agent definitions today, but the core architecture — the hierarchy model, issue specs, `.lgtm/` workspace, git conventions, and orchestration patterns — is agent-agnostic by design. The `.claude/` directory is the Claude Code implementation. The `.lgtm/` directory and the workflow model work with any agent that can read markdown and run git commands.

Future versions will add adapters for other agentic coding tools.

## Acknowledgments

LGTM builds on and adapts work from several open source projects:

| Project | What we adapted | Link |
|---------|----------------|------|
| **Blader humanizer** | AI writing pattern detection and removal. The humanizer skill and agent are based on this. | [blader/humanizer](https://github.com/blader/humanizer/) |
| **Wikipedia: Signs of AI writing** | The humanizer's pattern catalog draws from this community-maintained guide by WikiProject AI Cleanup. | [Wikipedia](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) |
| **Marketing skills** | Copywriting, copy-editing, content strategy, page CRO, SEO audit, AI SEO, and schema markup skills adapted from Corey Haines' collection. | [coreyhaines31/marketingskills](https://github.com/coreyhaines31/marketingskills) |
| **agent-browser** | Headless browser automation used by the QA and dogfood agents. | [anthropics/agent-browser](https://github.com/anthropics/agent-browser) |
| **GEO-SEO-Claude** | Reference implementation for geographic and AI search optimization patterns. | [zubair-trabzada/geo-seo-claude](https://github.com/zubair-trabzada/geo-seo-claude) |
| **claude-translation-skill** | Anti-fabrication checklist, terminology glossary structure, and i18n subagent patterns adapted for the translator skill. | [senshinji/claude-translation-skill](https://github.com/senshinji/claude-translation-skill) |
| **i18n subagent pattern** | The translator agent's architecture follows this subagent pattern for i18n specialist work. | [solberg.is/i18n-subagent](https://www.solberg.is/i18n-subagent) |
| **Agent Skills spec** | Skill file format follows the emerging agent skills standard. | [agentskills.io](https://agentskills.io) |

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## Security

See [SECURITY.md](SECURITY.md).

## License

[MIT](LICENSE)
