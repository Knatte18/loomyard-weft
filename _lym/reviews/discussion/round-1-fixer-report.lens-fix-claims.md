```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked every claim the fix added or rewrote against the code at HEAD. One is overstated (MEDIUM); every other claim holds.

**MEDIUM: D2's new bullet promises more than the untyped decode can deliver.**
- **Changed text:** `decision-record.md` D2, the new bullet: "The render decodes each group and entry, and skips one of the wrong YAML type … so a type error stays inside the button row. This keeps whole every load of `reedengine.LoadConfig` and of `desktophook.LoadConfig`". The D2 acceptance criterion repeats the claim.
- **The problem:** the layer loader checks each file's shape against the template before anything unmarshals, and that check compares only "mapping or not".
  - A `buttons` key that is not a mapping still fails the load.
  - So does a group written as a mapping, since the template holds a list there.
  - Decoding `buttons` as an untyped node never sees either error. Only a group that is the wrong non-mapping type, such as a scalar, stays inside the row.
- **Evidence:**
  - `internal/configengine/layers.go:345-347`: `ValidateMachine` refuses when `docIsMapping != (templateValue.Kind == yaml.MappingNode)`.
  - `layers.go:480` calls it on the load path.
  - `layers.go:296` makes the same shape refusal for the other layers.
  - `desktophook.afterWindow` (`internal/desktophook/window.go:44-48`) returns that load error for every window.

**Claims verified:**
- reed's server spawn strips `LYM_STRAND_NAME` and `LYM_PARENT` (`lifecycle.go:458`, `:660-671`). Panes get these variables only through a typed export (`panebin.go:37,40`), and reed sets no tmux environment variable for them. So "run-shell carries no strand name" and "a click reads as the operator" hold.
- `desktophook.LoadConfig` runs inside `afterWindow` (`window.go:44-48`).
- `PaneCwd` equals `AnchorPath` in the reed geometry (`hubgeom.go:89-90`).
- `NamedWorktree` returns a `Location` with `AnchorPath`, and it does not check that the worktree exists (`lymcwd.go:112-127`).
- `ResolveBattenPrime` refuses a beam that is being landed and names the worktree to re-run from (`refusal.go:80-88`).
- `lym shuttle open` resolves its location from its cwd (`shuttlecli/cli.go:76`), so the `cd {worktree}` in front of `agent` is needed.
- tmux's manual confirms that the `confirm-before -p` prompt takes status-left sequences.

**Out of scope:** the tmux 3.6 manual does not say that `run-shell -c` expands formats. The record already assumed it did with `'#{session_path}'` before this fix. `'#{@lym_button_dir}'` relies on the same assumption.

**Tool calls:**
1. Read `desktophook/window.go` and grepped reedengine for env injection, plus the `confirm-before` manual entry, to verify the `afterWindow` and env-strip claims and the D2 prompt claim. My context had no `afterWindow` body or tmux manual.
2. and 3. Read the `run-shell` manual entry for `-c` expansion. My context lacked it.
4. Grepped reedengine for `update-environment` and `ParentEnv`, to confirm nothing re-adds the stripped variables. My context lacked it.
5. Grepped configengine for where the shape check runs on load, to test D2's "keeps whole every load" claim. My context showed only `ValidateMachine` itself.
