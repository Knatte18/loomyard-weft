# Operator follow-ups: pause marker, shortcuts, stdin paste, hold end

Five operator follow-ups land together.

## What changes

- **Runs overview** (`lyx batten overview`, `internal/battencli`): a run a live pause covers shows `paused <minutes>` in its state cell, counted from the covering entry's `PausedAt`.
  A covered run that is `running`, `paused` or `starting` gets the marker; `blocked`, `awaiting` and `failed` keep their state.
  A run paused only by `lyx batten pause` or a pause boundary keeps the bare `paused`.
  The pause record is read once per poll; an unreadable record reads as no pause and is logged at Debug.
- **Chair held-turn notice** (`internal/seatengine`): the advisor-work predicate checks the advisor's session reading before its up line.
  An advisor reading `asking`, `idle-stalled`, `idle-done` or `dead` no longer defers the notice, including when its up line has not landed.
  The 10-minute cap stays as the backstop for an advisor wedged mid-turn.
- **Reed paste-submit** (`internal/reedengine`): the buffer is loaded with `load-buffer -` from the tmux client's stdin, chained with `paste-buffer` and the Enter in one invocation.
  A text over tmux's command size limit now pastes whole, where `set-buffer` failed the send with `command too long`.
  The trailing-`;` escape and the `--` guard are gone, since the text is no longer an argv element.
  The runner gains a stdin-writing variant that logs its args at Debug like the existing ones.
- **Operator shortcuts** (new `internal/shortcutcli`, mounted from `cmd/lyx`'s `newRoot`): `shortcuts.yaml`, embedded with `go:embed`, maps a name to one argv or a list of argvs.
  It ships `attach` → `reed attach` and `prime` → `orch start`, then `reed attach`.
  Each shortcut is a root command in the `Shortcuts` help group.
  It runs every argv through `clihelp.RunRootCtx` on a fresh root, so flags, envelopes and exit codes are the real command's, and a non-zero exit stops the sequence.
- **Loom status strand** (`internal/loomcli`): it takes at most an eighth of the free height instead of a quarter, with its minimum of 3 rows.
  A live strand from an older build is replaced at the next loom bootstrap.

No board issue is linked, so there is no `Closes` line.

## Design decisions to review

- Shortcut words pass verbatim onto the end of the last argv.
  The exception is the root's verbosity flag before the first `--`, which goes onto every argv, so `lyx -vv prime` reaches both `orch start` and `reed attach`.
  `--help`, `-h` and `--json` before `--` print the shortcut's own help and run nothing, so `--json` still never runs a command.
- A shortcut file that fails to parse mounts no shortcut and logs one Warn; a test in `cmd/lyx` keeps such a file out of a passing build.
  The test also fails on a shortcut named like a root command or another shortcut, an empty argv, and an argv that does not resolve to a runnable non-shortcut command.
- Shortcuts are not modules: the root-`Long` module check and the sandbox-coverage walk skip the `shortcutcli.GroupID` group, and `pattern/PATTERN-cli-cobra.md` lists `internal/shortcutcli` among the packages with a deviating shape.
- The held-advisor check reads the session state only, not the held-turn wait marker, which a held turn end already shows as `idle-stalled`.
- Text reaches tmux only as stdin bytes of `load-buffer -`, so no text can add or alter a command in the list.

## To check by hand

- The long-paste case in `internal/reedengine/paste_integration_test.go` (tag `tmux`) compiles but was not run; run `lyx gate test --tags tmux ./internal/reedengine` against a real tmux.
- `load-buffer -` relies on the tmux client forwarding stdin, checked on tmux 3.6; an older tmux is untested.
- The skip of the whole `shortcutcli.GroupID` group in the `cmd/lyx` guards (`clitree_test.go`, `sandbox_coverage_test.go`) lets through anything mounted in that group; confirm the file-versus-root group assertion in `cmd/lyx/shortcuts_test.go` bounds it.
- A global flag other than verbosity, such as a persistent flag added later, reaches only the last argv of a shortcut; confirm that is acceptable.
