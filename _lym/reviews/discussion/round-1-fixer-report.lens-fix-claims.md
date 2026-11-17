```yaml
lens: fix-claims
outcome: reported
```

fix-claims findings:

1. **MEDIUM.** The new D6 acceptance criterion reads "a regeneration with no workspace-content hook installed writes workspace files holding only `folders` and `settings`". This is false for beam files. Every beam workspace file also carries the top-level `"lym.generated": true` mark, written first, whatever hooks are installed.
   - `internal/fabricengine/hubworkspace.go:42` defines `beamWorkspaceMark = "lym.generated"`.
   - `hubworkspace.go:208` writes `markBeamWorkspace(beamContent)`.
   - `hubworkspace.go:227-230` prepends the key.
   
   The criterion holds only for the hub's own file. The fix could say "holding no member beyond those the regeneration writes today", or "no `tasks` key".

2. **LOW.** The new sentence in D6's Bound reads "the next regeneration or seed by an up-to-date binary restores them". Each writer restores only its own files, not all three:
   - a regeneration rewrites only the workspace files, which restores the `beam start` tasks;
   - only seed or spawn rewrites `tasks.json` and the keybindings block, which restores `board open` and the two keys (`internal/ideengine/spawn.go:228-243`, `internal/vscode/keybindings.go:172`).
   
   So after a regeneration alone, `board open` and the keys stay missing. The sentence should pair each writer with what it restores.

Checked and holding:
- `lymcwd.ResolveWithin` exists (`internal/lymcwd/lymcwd.go:143`).
- `InstallPairFolderNamer` is installed in `cmd/lym/main.go:175`. The ide hook installed beside it would run in every non-test process, because the install block is guarded only by `!testing.Testing()`.
- `ideengine.LymBinary` is `selfexec.LymBinary` (`spawn.go:37`).
- `vscode.WriteConfig` falls back to bare `lym` (`config.go:71-73`).
- The regeneration reads the prime's and the beams' anchors through `lymcwd.NamedWorktree` (`hubworkspace.go:106,144`).
- A tracked `tasks.json` skips the whole `.vscode` write, so the window lacks both `reed attach` and `board open` (`spawn.go:254-256`).
- A commit touching an unclaimed path goes to the cherry-picks (`replay.go:742`).
- Go's default HTML escaping writes `<`, `>` and `&`.
- That `"reveal": "silent"` brings the terminal forward on a non-zero exit is VS Code behaviour. No code in the repo can confirm or refute it.

Tool calls:
- One Bash call read `cmd/lym/main.go:140-160` and grepped `internal/vscode/keybindings.go`. My context did not have the condition guarding the hook install block, or how the keybindings block is replaced.
