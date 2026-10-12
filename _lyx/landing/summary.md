# Operator follow-ups: pause marker, hold end, stdin paste, shortcuts

Five small operator follow-ups, shipped together.

## What changes

- **Runs overview shows pauses.**
  `lyx batten overview` reads the pause record once per poll.
  A run a live pause covers (a slug entry, or a hub entry) and whose batten state is `running`, `paused` or `starting` shows `paused <minutes>` in its state cell, counted from the oldest covering entry and floored at zero.
  An uncovered batten-paused run keeps the bare `paused`; covered `blocked`, `awaiting` and `failed` runs keep their state.
  An unreadable record reads as no pause, logged at Debug, so the pane keeps drawing.
- **A held or dead advisor no longer defers the chair's held-turn notice.**
  The seat engine's advisor-work predicate skips an advisor whose session reads `asking`, `idle-stalled`, `idle-done` or `dead` before it looks at the up line.
  The 10-minute cap stays as the backstop for an advisor wedged mid-turn.
  Test rows now pin held and dead advisors.
- **Reed paste-submit sends a text of any length.**
  The buffer is loaded with `load-buffer -` from the tmux client's stdin, chained with `paste-buffer` and the optional Enter in the same single tmux invocation.
  The text is no longer an argv element, so the trailing-`;` escape and the `--` guard are gone, and no text can alter the command list.
  The tmux runner gains a run variant that writes given bytes to stdin and logs its args at Debug.
- **Top-level operator shortcuts.**
  The new `internal/shortcutcli` embeds `shortcuts.yaml`, which maps a name to one argv or a list of argvs.
  `lyx attach` runs `reed attach`; `lyx prime` runs `orch start`, then `reed attach`.
  Each shortcut is a root command in a `Shortcuts` help group and re-dispatches each argv through a fresh root, so output, flags and exit codes are the real command's; a non-zero exit stops the sequence.
- **The loom status strand takes at most an eighth of the free height** (was a quarter; minimum 3 rows unchanged).
  A live status strand from an older build is replaced at the next loom bootstrap.

Docs: the package docs of every touched package, `docs/overview.md` and the package-naming list in `pattern/PATTERN-cli-cobra.md` (which gains `internal/shortcutcli`).

## Design decisions

- Shortcuts pass trailing words, flags included, verbatim onto the last argv.
  The exception is the root's verbosity flag (`-v`, `-vv`, `--verbose`, `--verbose=<n>`) before the first `--`: it goes onto every argv wherever it sits, so `lyx -v prime` reaches both commands.
  `--help`, `-h` and `--json` before the first `--` print the shortcut's own help and run nothing, so `--json` never runs a command; after `--` everything is forwarded.
- A shortcut file that fails to parse mounts no shortcut and logs one Warn; a test in `cmd/lyx` keeps such a file out of CI.
  It also fails on a shortcut named like a root command or another shortcut, an empty argv, and an argv that names no runnable non-shortcut command.
- The sandbox-coverage walk and the root-`Long` module check skip commands by the shortcut help group.
  A command added to that group is skipped by both for that reason alone, so keep the group for shortcuts only.
- Shortcuts are not modules: a shortcut is an alias in the `PATTERN-cli-cobra` sense, stretched to a sequence of argvs.
- Hub, repo and user overrides are deliberately absent; changing a shortcut is a commit and a deploy.

## Check by hand

- The `tmux`-tier long-paste test (a text above tmux's command size limit pasted whole into a real pane) was added and compiled but not run.
  Run `lyx gate test --tags tmux ./internal/reedengine`.
- `load-buffer -` relies on the tmux client forwarding stdin; this was checked on tmux 3.6 only, and an older tmux is untested.
- Shortcut group skip: confirm the group-based skip in `cmd/lyx/clitree_test.go` and `cmd/lyx/sandbox_coverage_test.go` is acceptable, since it is not bounded to the shortcuts in the file.
- Global flags: a global verbosity flag reaches every expanded argv, but any other global flag reaches only the last argv.
  Try `lyx -v prime` and `lyx prime --json`.
