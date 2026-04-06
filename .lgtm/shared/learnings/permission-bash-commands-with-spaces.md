# Learning: Allowing Bash Commands with Spaces in the Executable Path

## Problem

The `/compare` skill runs VSCodium via:
```bash
"/c/Program Files/VSCodium/bin/codium" --diff "file1" "file2"
```

The executable path contains spaces (`Program Files`), so it must be quoted. Claude Code kept prompting for approval despite attempts to configure a `Bash()` permission allow rule.

## What Does NOT Work

### `Bash()` wildcard rules with quoted executables

```json
"Bash(\"/c/Program Files/VSCodium/bin/codium\" --diff *)"
```

The `*` wildcard does not match strings that contain spaces (like two quoted file paths). Even with the correct quote escaping, the allow rule never fired. The "Yes, don't ask again" option (option 2) proves this — it writes an exact-match rule with the full literal command including both file paths, not a wildcard.

### `PermissionRequest` hook with `permissionDecision`

```json
"hooks": { "PermissionRequest": [...] }
```

The `permissionDecision` field in `hookSpecificOutput` is **PreToolUse only**. Setting it in a `PermissionRequest` hook has no effect.

### Hook using `jq`

`jq` is not installed in this environment. Hooks using `jq -r '.tool_input.command'` fail silently (exit 0, no output), so the permission prompt still appears with no indication anything went wrong.

## What Works

### `PreToolUse` hook with Python-based JSON parsing

```json
"hooks": {
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "command",
          "command": "python -c \"import sys,json; d=json.load(sys.stdin); cmd=d.get('tool_input',{}).get('command',''); print('{\\\"hookSpecificOutput\\\":{\\\"hookEventName\\\":\\\"PreToolUse\\\",\\\"permissionDecision\\\":\\\"allow\\\"}}') if 'VSCodium' in cmd and '--diff' in cmd else None\""
        }
      ]
    }
  ]
}
```

**Why it works:**
- `PreToolUse` fires before the permission prompt
- `hookSpecificOutput.permissionDecision: "allow"` skips the prompt (this field is PreToolUse-only)
- Python is available; `jq` is not
- The hook reads stdin JSON, extracts `tool_input.command`, checks for `VSCodium` and `--diff`, and outputs the allow JSON only when both are present — otherwise outputs nothing, leaving normal permission checking intact

## Key Rules

| Rule | Detail |
|------|--------|
| `Bash()` wildcards don't match paths with spaces | `*` appears not to glob across spaces in file path arguments |
| `permissionDecision` is PreToolUse-only | `PermissionRequest` hooks cannot auto-allow commands |
| `jq` is not installed | Use `python` for JSON parsing in hooks |
| Hook silent failure looks identical to hook success | Always pipe-test the hook command before trusting it |

## Pipe-Test Command

To verify a hook before deploying:

```bash
echo '{"tool_name":"Bash","tool_input":{"command":"\"/c/Program Files/VSCodium/bin/codium\" --diff \"file1\" \"file2\""}}' | python -c "import sys,json; d=json.load(sys.stdin); cmd=d.get('tool_input',{}).get('command',''); print('{\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"allow\"}}') if 'VSCodium' in cmd and '--diff' in cmd else None"
```

Expected output:
```json
{"hookSpecificOutput":{"hookEventName":"PreToolUse","permissionDecision":"allow"}}
```
