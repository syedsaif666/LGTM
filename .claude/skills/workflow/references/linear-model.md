# Linear Model

How Linear's hierarchy maps to our filesystem and GitHub.

## The mapping

| Linear primitive | Our mapping | GitHub mapping |
|-----------------|-------------|----------------|
| Initiative | The repo itself | — |
| Project | `.lgtm/shared/plans/p{N}-{slug}/PLAN.md` | GitHub milestone |
| Milestone | Section in PLAN.md or sub-plan | Encoded in issue ID prefix (`P2M1`) |
| Issue | `.lgtm/shared/plans/{project}/issues/P{N}M{M}-{NNN}.md` | PR (linked via issue ID in title) |

## The contract

**PR title starts with the issue ID.** That's the link between the filesystem
and GitHub.

- Issue file: `P2M1-003.md`
- PR title: `P2M1-003: Add rate limiting middleware`
- Branch name: `auth-rate-limiting` (descriptive, not required to contain issue ID)

The PR title is the source of truth for linking. Branch names are for humans.

## Why GitHub milestones = projects

GitHub milestones group PRs. Our projects group issues. The mapping is 1:1.

```bash
# Create milestone
gh api repos/{owner}/{repo}/milestones --method POST -f title="{project name}"

# Attach PR to milestone
gh pr edit {pr-number} --milestone "{project name}"
```

Every PR in a project gets attached to the same milestone. When all PRs merge,
the milestone is complete.

## Why not GitHub Issues?

Our "issues" are agent-executable spec files, not conversation threads.
They live in the filesystem so agents can read them as cold-start specs.
GitHub Issues don't support structured frontmatter, test cases, or
dependency DAGs in a machine-readable format.

The PR is the GitHub artifact. The issue file is the spec. They link
through the PR title.
