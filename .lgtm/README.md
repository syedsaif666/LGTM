# .lgtm/ — Workspace

Everything the initiative produces and tracks lives here. The `.claude/`
directory holds the agent framework (definitions, skills, rules). This
directory holds the work itself.

## Structure

```
.lgtm/
  STATUS.md                            the initiative dashboard
  ai/                                  agent-owned space
    logs/                              daily activity logs
      YYYY-MM-DD/
        LOG.md
    process-artifacts/                 all agent output, organized by hierarchy
      {project}/
        {milestone}/
          {issue}/
  shared/                              collaboration space (agents + humans)
    plans/                             project plans and issue specs
      p{N}-{slug}/
        PLAN.md
        issues/
          P{N}M{M}-{NNN}.md
    learnings/                         lessons, gotchas, patterns discovered during work
    resources/                         reference data, seed files, schemas, assets
  user/                                human-owned space
    backups/                           editor configs, environment snapshots
    _archive/                          retired plans, deprecated agents, old work
```

## The three zones

### `ai/` — Agent space

Agents write here. Humans read here.

**`ai/logs/`** — The `log-entry.py` script appends timestamped entries to
`YYYY-MM-DD/LOG.md` whenever something happens: a PR merges, a decision
is made, a milestone completes. These survive context compaction and give
any future session a trail of what happened and when.

**`ai/process-artifacts/`** — Every agent writes its output here, organized
by the initiative hierarchy (project / milestone / issue). What goes here
depends on the work:

| Initiative type | Example artifacts |
|----------------|-------------------|
| Software project | Generated code review reports, test results, migration scripts |
| Content pipeline | Enriched JSON entries, rewritten copy, translation files |
| Workflow design | Validated JSON workflows, integration test reports |
| Research | Analysis reports, data extracts, comparison documents |

Artifacts are never deleted. They are the persistent record of what each
agent produced. Downstream agents or humans can read from this folder later.

### `shared/` — Collaboration space

Both agents and humans read and write here.

**`shared/plans/`** — Project plans and their issue specs. A plan defines
milestones and their dependency DAG. Issue specs are self-contained
documents that an agent or human can execute without reading the plan.
See `AGENTS.md` for the hierarchy and naming conventions.

**`shared/learnings/`** — Knowledge captured during work. When an agent
or human discovers something worth remembering (a workaround, a gotcha,
a pattern), it goes here. Future sessions can reference these.

**`shared/resources/`** — Reference material that feeds into the work.
Seed data, schemas, brand assets, API response samples, templates —
anything that multiple agents or sessions need access to.

### `user/` — Human space

Humans write here. Agents generally don't touch this.

**`user/backups/`** — Environment configurations, editor settings, anything
the human wants to preserve alongside the initiative.

**`user/_archive/`** — Retired work. Old plans, deprecated agent configs,
superseded content. Moved here rather than deleted so nothing is lost.

## STATUS.md

The single dashboard for the initiative. Initiative health, project
progress table, active work, latest update, and a table for unplanned
contributions. Read it to understand where things stand. Updated by the
orchestrator after each meaningful event (PR merge, milestone complete,
health change).
