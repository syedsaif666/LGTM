# {CONFIGURE: Project name}

## Initiative

{CONFIGURE: Initiative goal — one-line description}

## Key Dependencies

Source of truth: `package.json` (or project equivalent).

| Package | Version | Notes |
|---------|---------|-------|
| ... | ... | ... |

## Build & Dev

{CONFIGURE: Build and dev commands}

Framework utilities:
- `python .claude/scripts/now.py` — current timestamp (millisecond precision)
- `/tree [path]` — project folder structure

## Hierarchy

Full workflow: `.claude/skills/workflow/SKILL.md`.
Hierarchy reference: `.claude/skills/workflow/references/hierarchy.md`.

| Level | Filesystem | GitHub |
|-------|-----------|--------|
| Initiative | The repo | — |
| Project | `.lgtm/shared/plans/p{N}-{slug}/PLAN.md` | GitHub milestone |
| Milestone | Section in PLAN.md or sub-plan | Encoded in issue ID prefix |
| Issue | `.lgtm/shared/plans/{project}/issues/P{N}M{M}-{NNN}.md` | PR (issue ID in title) |

The issue-to-PR contract: PR title starts with the issue ID
(e.g., `P2M1-001: short description`). Branch names are descriptive.
Each project gets a `{project}-base` branch; issue branches target it.

## Agents

- **Orchestrator** (`.claude/agents/orchestrator.md`) — drives the plan → issue → PR lifecycle
- **Issue writer** (`.claude/agents/issue-writer.md`) — creates issue files from plans

## Projects

Plans: `.lgtm/shared/plans/{project}/PLAN.md`.
Status: `.lgtm/STATUS.md` (read on demand, not auto-loaded).
Daily logs: `.claude/logs/YYYY-MM-DD/LOG.md`.

## Conventions

- Use `python` not `python3` (Windows environment)
- All scripts are Python — `.claude/scripts/*.py`
- No Co-Authored-By or AI attribution in commits
- Never `git add -A` — stage specific files only
- PRs only — never commit directly to master
- One branch per issue, one PR per issue — PR title must start with issue ID
- PR title is the issue-to-PR contract: `P{N}M{M}-{NNN}: short description`
- Dates in STATUS.md and LOG.md use human-friendly format: "Apr 5, 2026"
  (tables), full day name in log headers. YYYY-MM-DD for folder names.
- Timestamp all log entries and status updates using `python .claude/scripts/now.py`
