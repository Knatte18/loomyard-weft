```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code and tmux at HEAD.

**MEDIUM: card 1's `ChangeDirToWord`, and card 9's click line built through it, are wrong for the pwsh dialect.**
- Changed text: card 1 says "`Set-Location -LiteralPath &lt;word&gt;` in pwsh" and "The click line passes a `#{q:...}` format that tmux expands into a quoted word at the click". Card 9 composes `sh.ChangeDirToWord("#{q:@lym_button_dir}")` with `sh` being `shell.ForGOOS()`, which is pwsh on Windows.
- What's wrong: `#{q:}` quotes the POSIX way, with backslashes. pwsh does not read backslash escapes, so the expanded word is not one quoted pwsh word.
- Evidence: a probe on tmux 3.6 expanded `#{q:@d}`, with `@d` set to `C:\a b\it's#x`, to `C:\\a\ b\\it\'s\#x`. Under pwsh that splits at the space and keeps the doubled backslashes, so `Set-Location` fails for any Windows path. Reed does support psmux sessions with run-shell (`internal/reedengine/doc.go:105,110`). The follow line's `#{q:session_name}` gets by only because session names carry no backslash or space.

**LOW: card 4's new reason sentence says `ParentOf` checks the branch "before it reads `parent_worktree`".**
- Evidence: `internal/hubgeom/parentof.go:33` tests `origin.ParentWorktree == ""` first. The branch comparison (`:42`) comes after it and before the function returns `parent_worktree`.
- The reason is right; the order it states is not. It should say `ParentOf` answers the prime before it returns `parent_worktree`.

Claims that hold:
- **Card 9's failed-directory-change claim:** on tmux 3.6, a `run-shell -b` line whose `cd` fails and that then exits 0 left `pane_in_mode` at 0. Stderr output with exit 0 opens no view mode either, so the claim that view mode stays shut still holds.
- **Card 9's probe description** matches the probe the fixer ran.
- **Card 5:** `TestSmokeSelvage` (`smoke_selvage_keepalive_test.go:41`) contains the subtest `StatusLinePinsIdentityAndPosition` (`:278`), and its comment at `:272-273` says the two-line bar.
- **Card 8:** "`tmuxQuote` doubles every `#` of the line, the directory's included" matches `tmuxQuote` as card 8 defines it.

Note: my probes created and killed a tmux server on `/home/knatte/Code/loomyard-LYMHUB/reed-status-buttons/.scratch/f.sock`, so that socket file may be left behind. As a read-only fork I did not remove it.

Tool calls:
- tmux probe of view mode after a failed `cd`, and the empty-directory case; my context had no view-mode result for a line whose `cd` fails.
- tmux probe of view mode with correct window targets; the first probe targeted a window that did not exist.
- tmux probe of a failed `cd` against stderr-only output; this separated the two causes of view mode.
- tmux probe of `#{q:}` on a Windows path, plus a grep of `doc.go` for psmux and run-shell; my context lacked how `q:` quotes and whether reed's run-shell lines run on Windows.
