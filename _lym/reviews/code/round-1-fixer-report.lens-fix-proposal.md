No findings.

Each proposal I challenged held up:
- **F1:** the by-slug children carry `--no-prime-claim` and the `--all` run is untouched. I found no case where the flag blocks a claim that should happen.
- **F2:** I found no other consumer of the start envelope, so renaming the hub's `shortname` key to `name` and moving `worktrees` up to the result breaks nothing.
- **F3:** only the dobby outcomes refused and failed turn a worktree into a failure.
- **F10:** the shell-special check on the lym path and shortname fits the WSL launcher line.
- **F11:** the lock file lands under `<state>/lym/<hash8>/launchers.lock`, and the other users of `UserStateDir` (slackcli, configreg, machinecli hubs) take only the directory itself. No sweeper of the state directory's children turned up.

Minor note, below LOW because nothing breaks: the F9 refusal text says "no live hub reads the shortname" for any `lookupHub` failure, including an unreadable record. The appended cause still names the real reason.

Tool calls (2 of 6):
1. `grep UserStateDir()` across non-test Go files — to see whether anything enumerates or sweeps the state directory the F11 lock file lives in; my context did not show other users.
2. `grep` for files naming `machine start` or `machinecli` outside the package — to find consumers of the start envelope's renamed and moved keys; my context did not show any.
