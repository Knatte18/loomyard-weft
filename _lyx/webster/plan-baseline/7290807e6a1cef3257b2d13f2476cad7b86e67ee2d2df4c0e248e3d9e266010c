# Card 7 — socket-path-refused-at-boot

**Create:**
- `internal/reedengine#socketBaseFromEnv`
- `internal/reedengine#socketDirUnder`
- `internal/reedengine#resolvedSocketDir`
- `internal/reedengine#unixSocketPathLimit`
- `internal/reedengine#checkSocketPathLength`

**Edit:**
- `internal/reedengine#socketDirFromEnv`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine/socketfile_test.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decision D6: on POSIX the pre-tmux block of the boot path refuses a socket path over the OS's limit, naming `TMUX_TMPDIR` as the way forward.

`socketDirFromEnv` keeps its contract and is split into its two halves: `socketBaseFromEnv` answers `$TMUX_TMPDIR` else `/tmp`, and `socketDirUnder` joins `tmux-<uid>` onto a base; `socketDirFromEnv` composes the two, so the directory rule is still spelled once in `socketfile.go`.
`resolvedSocketDir` resolves the base's symlinks with `filepath.EvalSymlinks`, keeping the unresolved base when resolution fails, and joins the per-user directory, so a symlinked `TMUX_TMPDIR`, or macOS's `/tmp` resolving to `/private/tmp`, is measured at the length tmux sees.
`unixSocketPathLimit` answers 107 for `linux`, whose `sun_path` is 108 bytes, and 103 for every other value, macOS and the BSDs among them, whose `sun_path` is 104; it takes the GOOS as a parameter so the table test covers both.
`checkSocketPathLength` is pure: it joins `dir` and `key`, admits a path of at most `limit` bytes, and refuses a longer one with an error naming the path, its byte length, the limit and the way forward, "set `TMUX_TMPDIR` to a shorter directory".

`ensureServerAndSessionLocked` calls `checkSocketPathLength(resolvedSocketDir(), e.Socket(), unixSocketPathLimit(runtime.GOOS))` right after `validateBootConfig` and before `probeCapabilityLocked`, on every GOOS but `windows`, where psmux keeps no socket file, and returns the refusal.
Bound: the check runs on the boot path only; a verb run later under a longer `TMUX_TMPDIR` than the server booted with still gets tmux's own error.
`doc.go`'s socket-file sentence under the multiplexer contract gains the boot check and its bound.

Tests: a new `TestBootSocketPath` in `socketfile_test.go` with three steps: the limit per GOOS; `checkSocketPathLength` admitting a path at the limit and refusing one byte over with a message naming `TMUX_TMPDIR`, the path and both lengths; and `resolvedSocketDir` under a `TMUX_TMPDIR` set with `t.Setenv` to a short symlink pointing at a longer directory under `t.TempDir()`, answering the per-user directory under the symlink's target.
`TestUp_BootValidation` gains the row `SocketPathTooLong`, which sets `TMUX_TMPDIR` to a directory whose per-user path plus the engine's key exceeds the Linux limit and expects the refusal naming `TMUX_TMPDIR` with zero tmux calls recorded; the row is skipped on Windows.
The `internal/reedengine` row in `test-budget.yaml` rises by one, to 244.

**ImpactSummary:** One new pre-tmux refusal on the boot path and a two-way split of the socket directory rule; every other caller of `socketDirFromEnv` sees the same answer, and only a hub whose socket path is over the limit, which could never boot before, is refused with a named remedy.
