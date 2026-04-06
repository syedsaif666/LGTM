# Hierarchy Reference

How the mental model maps to the filesystem, git, and GitHub.

## Levels

| Level | What it is | Where it lives | Git mapping |
|-------|-----------|----------------|-------------|
| **Initiative** | The overarching goal | The repo itself | — |
| **Project** | Self-contained body of work | `.lgtm/shared/plans/p{N}-{slug}/PLAN.md` | GitHub milestone + `{project}-base` branch |
| **Milestone** | Checkpoint within a project | `.lgtm/shared/plans/{project}/m{N}-{name}.md` (if large) or section in PLAN.md | Group of issue PRs on the base branch |
| **Issue** | Atomic unit of work | `.lgtm/shared/plans/{project}/issues/P{N}M{M}-{NNN}.md` | One branch + one PR → base branch |
| **Commit** | Documents WHY a change was made | Git history | — |

## Naming conventions

| Entity | Pattern | Example |
|--------|---------|---------|
| Plan | `.lgtm/shared/plans/p{N}-{slug}/PLAN.md` | `p2-auth-system/PLAN.md` |
| Milestone sub-plan | `.lgtm/shared/plans/p{N}-{slug}/m{N}-{name}.md` | `p2-auth-system/m1-oauth.md` |
| Issue file | `.lgtm/shared/plans/p{N}-{slug}/issues/P{N}M{M}-{NNN}.md` | `P2M1-001.md` |
| Base branch | `{project}-base` | `auth-base` |
| Issue branch | `{project}-{descriptive-slug}` | `auth-oauth-callback` |
| PR title | `P{N}M{M}-{NNN}: short description` | `P2M1-001: Add OAuth callback handler` |

**Issue ID format:** `P{project}M{milestone}-{sequence}` (e.g., `P2M1-003`)

## Filesystem layout

```
.claude/                                 ← agent framework (definitions, skills, rules)
  logs/
    YYYY-MM-DD/
      LOG.md                           ← daily log entries
  agents/
    orchestrator.md                    ← drives the workflow
    issue-writer.md                    ← creates issues from plans
  skills/
    workflow/                          ← this skill
  rules/
    agent-protocol.md                  ← universal agent constraints
    git-workflow.md                    ← git safety rules

.lgtm/                                  ← workspace (plans, artifacts, status, resources)
  STATUS.md                            ← current initiative status
  shared/
    plans/
      p2-auth-system/
        PLAN.md                        ← project plan (milestones, DAG, context)
        m1-oauth.md                    ← milestone sub-plan (if scope warrants)
        issues/
          P2M1-001.md                  ← issue spec (agent or human)
          P2M1-002.md
  ai/
    process-artifacts/                 ← all agent output, mirroring project hierarchy
```

## Flow

```
Plan → Issues → Execution → PRs → Milestone complete → Project complete
```

1. Plan defines milestones and their DAG
2. issue-writer agent reads plan, creates issue files
3. Open questions in issues resolved before status → `ready`
4. Agent or human executes each issue (reads only its issue file)
5. Each issue = one branch + one PR targeting the project base branch
6. All milestone issues merge → milestone complete → update STATUS.md
7. All milestones complete → project base branch PR → develop → master
