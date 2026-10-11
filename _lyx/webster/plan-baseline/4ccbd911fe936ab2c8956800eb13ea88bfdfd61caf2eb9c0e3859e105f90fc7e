# Card 2 — tmux-conf-path-validation

**Create:**
- `internal/reedengine#tmuxConfEnv`
- `internal/reedengine#tmuxConfPath`
- `internal/reedengine#TestTmuxConfPath`

**Edit:**
- `internal/reedengine#Engine.preflightBootHost` -> `func (e *Engine) preflightBootHost() (shellPath, tmuxConf string, err error)`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#Engine.withBootOpLockKeepingZoom`
- `internal/reedengine#TestUp_BootValidation`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#resolveShellPath`

**Intent:** Validate a non-empty `tmux_conf` on every POSIX boot, at the point and in the way an unresolvable `shell` is refused today (D2), and leave Windows unvalidated (D7).

`tmuxConfPath` and `tmuxConfEnv` live in the new file `internal/reedengine/tmuxconf.go`, whose header comment names it the home of the operator tmux config feature.
`tmuxConfPath` returns the empty string for an empty `raw`.
Otherwise it replaces a leading `~/` with `home` and then refuses, in this order: a path that is not absolute, a path holding a whitespace rune or any of `*`, `?`, `[`, `\`, `'`, `"`, `;`, `#`, `{`, `}` and `$`, a path `os.Stat` cannot find, and a path that is not a regular file.
Every error names the key `tmux_conf`, the variable `tmuxConfEnv` and the path, and reads like `resolveShellPath`'s `key shell` error, so one refusal style covers both keys; it never names tmux's parser rules beyond the character set.
The character set is refused rather than quoted because `source-file` treats its argument as a glob and card 3 nests the path in a command string tmux parses a second time; a refusal is one check with a clear error.

`preflightBootHost` returns the validated path as a second result.
It keeps its Windows early return, so Windows parses the key and checks nothing.
On POSIX it reads the home directory through `os.UserHomeDir`, passing the empty string on error so a `~/` path then fails the absolute check naming the path, and calls `tmuxConfPath` after `resolveShellPath`.
`ensureServerAndSessionLocked` and `withBootOpLockKeepingZoom` take the new result; this card discards it in both, and card 3 consumes it in the boot.

`TestTmuxConfPath` is table-driven over a `t.TempDir` fixture: the empty value, a `~/` value expanding against the home argument to an existing file, an absolute existing regular file accepted unchanged, and one refusal row each for a relative path, a missing file, a directory, a name holding `[`, a name holding a space, and a name holding `$`; every refusal row asserts the error names `tmux_conf`, `LYX_REED_TMUX_CONF` and the path.
`TestUp_BootValidation` gains two rows, both setting the relative path `tmux.conf`, which needs no fixture file.
`TmuxConfRelative` skips on Windows in `setup` the way `ShellUnresolvable` does, with `wantErr` naming the key, so the refusal's wiring is pinned at both boot entries with zero tmux calls; `TestTmuxConfPath` stays the covering test for each refusal class, as it does for `shell`'s single `ShellUnresolvable` row.
`TmuxConfUnvalidatedOnWindows` skips off Windows, with `notErr` naming the key the way `ValidWatchdog_On` does, so the Windows boot is not refused for `tmux_conf` (D7's "neither validated" half).
`test-budget.yaml` raises `internal/reedengine` to 248 for the new test.

`doc.go`'s "Operator config" paragraph gains one sentence: a non-empty `tmux_conf` is refused on a POSIX boot unless it names an absolute path to an existing regular file free of whitespace and the listed characters, naming the key, the variable and the path; card 3 rewrites the paragraph around the sourcing itself.

**ImpactSummary:** Re-signs one unexported boot pre-flight with two callers in the package; an empty key changes nothing, and a bad key now fails `lyx reed up` and every booting op before any tmux round trip.
