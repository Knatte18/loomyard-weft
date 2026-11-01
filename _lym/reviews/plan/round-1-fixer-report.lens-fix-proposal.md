```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged the proposals the fixer carried out for F1 through F7.

Proposals checked: F1 (card 9 click line, card 8 confirmed line, card 1 `ChangeDirToWord`), F2, F3, F4, F5, F6, F7.

**MEDIUM: F1's quoted-word directory change is wrong in the pwsh dialect**
- **Text concerned:** card 1's new `ChangeDirToWord` bullet ("`word` being already one quoted shell word … `Set-Location -LiteralPath &lt;word&gt;` in pwsh"), and card 9's `sh.ChangeDirToWord("#{q:@lym_button_dir}")`.
- **Problem:** `composeButtonCommand` takes `sh`, which is `shell.ForGOOS()`, so on Windows the click line is `Set-Location -LiteralPath #{q:@lym_button_dir}`. tmux's `q:` modifier escapes for POSIX sh, putting a backslash before each space and special character. pwsh does not read a backslash as an escape.
- **Concrete case:** a run directory such as `C:\Users\a b\repo` expands to `C:\\Users\\a\ b\\repo`. pwsh splits that into two arguments, so `Set-Location` fails and the verb starts in the server's directory.
- **What differs from before:** the follow line has the same `#{q:...}` mismatch, but only over `session_name`, which holds no spaces. A directory path routinely does.
- **What would fix it:** either card 1's pwsh form states that it gets a POSIX-quoted word and why that is accepted (for example the windows-status-line-is-an-unbranched-accepted-degrade decision), or the pwsh form stays unclaimed.
- **Evidence:** card 1's pwsh bullet as changed, card 9's `composeButtonCommand` bullet, and `internal/reedengine/panebin.go:128-133`, where the follow line uses `#{q:session_name}` under the same `sh`.

**A case I built that turned out harmless.** When the directory change fails (a run directory removed after the render), it does not open view mode, even though its stderr is not discarded.
- I probed tmux 3.6 with no client attached. `run-shell -t p:0 'cd /no/such/dir; exit 0'` left `#{pane_in_mode}` at `0`, while adding `echo to-stdout` to the same line set it to `1`.
- So in that probe run-shell showed stdout but not stderr, and card 9's claim that the failed change "leaves the verb in the server's directory" holds without opening view mode.

The other proposals hold:
- **F3:** `ParentOf` compares `parent_branch` only against the main worktree's branch, so a branch no worktree holds reaches the `parent_worktree` path.
- **F4:** card 3 already limits `{strand}` to `window: agents` entries, so the narrower flag never marks a plain-row entry.
- **F5:** the moved directory check also runs ahead of the confirmed call's hash check, which is harmless.
- **F2, F6 and F7:** I found no case where the proposal is wrong.

Tool calls beyond my context:
- tmux probe of `run-shell -b` with a failing `cd`: the context lacked whether run-shell surfaces stderr. The result was inconclusive because the background job raced the read.
- Foreground `run-shell` with a failing `cd`, with and without `2&gt;/dev/null`: same question, without the race.
- The same command refused by the rm hook, because I removed the socket by a variable path. It did not run.
- Foreground `run-shell` with `echo` plus a failing `cd`: to confirm stdout opens view mode, so the earlier `0` meant stderr was dropped.
