# Editor Config Backup

VSCodium + Claude Code terminal workflow configuration.
These files set up auto-launching Claude Code sessions, keybindings, font/ligatures, and terminal behavior.

## Files

| File | Restore to | Purpose |
|------|-----------|---------|
| `claude-keybindings.json` | `C:\Users\Duke\.claude\keybindings.json` | Shift+Enter for newline, Alt+E for external editor |
| `vscodium-settings.json` | `C:\Users\Duke\AppData\Roaming\VSCodium\User\settings.json` | Font (FiraCode Nerd Font), ligatures, persistent sessions, hide default terminal, tab styling, sidebar right |
| `vscodium-keybindings.json` | `C:\Users\Duke\AppData\Roaming\VSCodium\User\keybindings.json` | Pass Shift+Enter through to Claude Code in terminal, disable Ctrl+Shift+W (close window) |
| `.vscode/tasks.json` | Already in repo at `D:\STL\silverthreadlabs\.vscode\tasks.json` | Auto-launch terminal tabs on folder open |

## Terminal Tabs (tasks.json)

Auto-launches on folder open via "Autostart" (waits 5s for extensions, then launches all):

| Tab | Icon | Color | Command |
|-----|------|-------|---------|
| AI - Prime | robot | red | `claude --resume` |
| AI - Knight | shield | green | `claude --resume` |
| Usage | graph | blue | `claude` (type `/usage` once loaded) |
| AI - Spawn | robot | yellow | `claude` (manual run only via Run Task) |

Additional tasks:
- **Launch All** — runs Prime, Knight, Usage in parallel
- **Autostart** — runs on folder open (Wait for Extensions -> Launch All)

## Font & Ligatures

- Editor: FiraCode Nerd Font with ligatures enabled (working)
- Terminal: FiraCode Nerd Font with `terminal.integrated.fontLigatures: true` (broken upstream since VSCode 1.108, tracked at microsoft/vscode#286742 — setting left in place for when the fix ships)

## Known Issues

- Terminal ligatures broken in VSCode/VSCodium 1.108+ due to broken `@xterm/addon-ligatures` package ([microsoft/vscode#286742](https://github.com/microsoft/vscode/issues/286742))

## Migrating to VS Code or Cursor

Copy the `vscodium-*.json` files to the target editor's config folder:

| Editor | Path |
|--------|------|
| VSCodium | `C:\Users\Duke\AppData\Roaming\VSCodium\User\` |
| VS Code | `C:\Users\Duke\AppData\Roaming\Code\User\` |
| Cursor | `C:\Users\Duke\AppData\Roaming\Cursor\User\` |

Rename `vscodium-settings.json` to `settings.json` and `vscodium-keybindings.json` to `keybindings.json` when restoring. `claude-keybindings.json` and `.vscode/tasks.json` work everywhere as-is.
