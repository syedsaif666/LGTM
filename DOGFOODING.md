# Dogfooding Architecture

LGTM is both the framework source and a project that consumes itself.
This document defines how the two roles coexist without collision.

**Status:** Planned. Today the source lives in `.claude/`. This plan
migrates it to root level.

## The Problem

Claude Code reads from `.claude/`. But if the source also lives there,
every consumer repo's `.claude/` becomes a fork — edits to the framework
get made in the wrong place, and upstream never sees them.

## The Solution

Root level = distributable source. `.claude/` = configured instance.
A sync script bridges them. Two hooks keep everyone honest.

```
LGTM/
├── agents/              <- SOURCE (distributable, tracked by git)
├── skills/
├── scripts/
├── rules/
├── CLAUDE.md
├── AGENTS.md
├── .claude/             <- INSTANCE (configured, gitignored)
│   ├── agents/
│   ├── skills/
│   ├── scripts/
│   ├── rules/
│   ├── CLAUDE.md
│   └── settings.json
└── .lgtm/               <- workspace
```

## Two Types of Files

**Identical files (~43 of 50+):** Skills, scripts, rules, references.
Byte-for-byte the same between source and instance. The sync script
copies these directly.

**Configured files (~7):** Files containing `{CONFIGURE:}` placeholders
in the source, filled with project-specific values in the instance.
Copied on first sync only, protected from overwrite on subsequent syncs.

Known configured files:
- `AGENTS.md`
- `CLAUDE.md`
- `agents/fullstack-engineer.md`
- `agents/content-editor.md`
- `skills/test-strategy/SKILL.md`
- `skills/translator/SKILL.md`
- `.lgtm/STATUS.md`

## Three Moving Parts

### 1. Sync Script (`scripts/sync.py`)

Copies root source -> `.claude/`, respecting the protect list.

- **First run:** copies everything, including configured files
- **Subsequent runs:** copies identical files, skips protected files
- **Force mode:** `--force` overwrites protected files (reset instance)

### 2. PostToolUse Hook — Auto-Sync

When Claude edits a root source file, this hook auto-runs the sync
script so `.claude/` stays current without manual intervention.

Defined in `.claude/settings.json`:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "<script that checks if edited file is a root source file, and if so runs python scripts/sync.py>"
          }
        ]
      }
    ]
  }
}
```

Reference: https://code.claude.com/docs/en/hooks — PostToolUse receives
`tool_input.file_path` on stdin as JSON, exit 0 to proceed silently.

### 3. PreToolUse Hook — Source Guard

When Claude tries to edit a file inside `.claude/`, this hook denies
the edit with a graceful message pointing to the root source file.

Defined in `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "<script that checks if target path is inside .claude/, denies with JSON permissionDecision + systemMessage pointing to root equivalent>"
          }
        ]
      }
    ]
  }
}
```

Allowlist for instance-only files (no root equivalent):
- `.claude/settings.json`
- `.claude/settings.local.json`
- `.claude/.onboarded`

Reference: https://code.claude.com/docs/en/hooks — PreToolUse can return
`permissionDecision: "deny"` with a `permissionDecisionReason` via JSON
on stdout (exit 0). This shows a clean message, not a generic error.

## Search Steering

`.claude/` is added to `.gitignore`. ripgrep respects `.gitignore` by
default, so Claude Code's Grep tool naturally skips instance files and
returns root source files only. One entry, two jobs (git exclusion +
search exclusion).

## Implementation Steps

### Step 1: Create root source tree

Copy the current `.claude/` contents to root level:
- `agents/`, `skills/`, `scripts/`, `rules/`
- `CLAUDE.md`, `AGENTS.md`

These become the tracked source. Do not delete `.claude/` yet.

### Step 2: Build `scripts/sync.py`

The sync script must:
- Walk the root source tree
- Copy each file to its `.claude/` equivalent
- Skip files on the protect list if they already exist in `.claude/`
- Support `--force` to overwrite protected files
- Print what it copied and what it skipped

### Step 3: Add `.claude/` to `.gitignore`

This makes the instance untracked and invisible to ripgrep.

### Step 4: Build the PreToolUse guard hook

A Python script (e.g. `scripts/source-guard.py`) that:
- Reads JSON from stdin
- Extracts `tool_input.file_path`
- If path is inside `.claude/` and not on the allowlist:
  - Outputs JSON with `permissionDecision: "deny"` and a
    `permissionDecisionReason` like "Edit the source at
    agents/orchestrator.md instead — it will sync to .claude/ automatically"
  - Exits 0
- Otherwise exits 0 silently

Register it in `.claude/settings.json` as a PreToolUse hook.

### Step 5: Build the PostToolUse auto-sync hook

A Python script (e.g. `scripts/auto-sync.py`) that:
- Reads JSON from stdin
- Extracts `tool_input.file_path`
- If path is a root source file (under `agents/`, `skills/`, etc.):
  - Runs the sync script for that file
  - Exits 0
- Otherwise exits 0 silently

Register it in `.claude/settings.json` as a PostToolUse hook.

### Step 6: Run initial sync and verify

- Run `python scripts/sync.py` to populate `.claude/` from root
- Confirm Claude Code still reads `.claude/CLAUDE.md` and operates normally
- Test: edit a root source file, confirm `.claude/` copy updates
- Test: try to edit a `.claude/` file, confirm graceful denial

### Step 7: Clean up git history

- Remove `.claude/` tracked files from git (they're now gitignored)
- Commit the root source tree as the canonical source

## Why Not Symlinks?

- Windows requires Developer Mode + git config changes
- Git on Windows defaults to `core.symlinks = false`, silently copies
- Symlinks cannot handle files that differ between source and instance
- Cross-platform fragility outweighs the convenience

## Why Not chezmoi/Stow/yadm?

- chezmoi is the closest fit but adds a tool + templating language for
  a problem solvable with a focused script
- GNU Stow has no template support and poor Windows compatibility
- yadm assumes home-directory topology, wrong for this use case
- Nix Home Manager has no Windows support
