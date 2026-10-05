# Card 19 — Reed removes the socket file of a server it tore down

**Create:**
- `internal/reedengine/socketfile.go`
- `internal/reedengine/socketfile_test.go`

**Edit:**
- `internal/reedengine#Engine.ensureServerGoneLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine/contract_integration_test.go`

**Uses:**
- `internal/reedengine#Engine.reapSocketProcesses`
- `internal/reedengine#Engine.Down`

**Intent:**
When reed's teardown kills its server and the socket file stays behind, reed removes that file after confirming the server is gone (D9).
`socketfile.go` holds an unexported helper that, given the `-L` key, resolves the socket path the way tmux does (`$TMUX_TMPDIR`, else `/tmp`, then `tmux-<uid>/<key>`) and removes it only when all of these hold:
the path is a socket (`os.Lstat`);
a `net.Dial("unix", path)` fails, so no server answers;
and a second `os.Lstat` reports the same file (`os.SameFile`) as the first.
A successful connection, a non-socket path, or a changed identity leaves the file alone, and a missing file is no error.
It is a no-op on Windows, where psmux keeps no socket file.
`ensureServerGoneLocked` calls it once no tmux process remains on the socket, on both its paths (the graceful exit and the `reapSocketProcesses` force-reap); `Down`'s `kill-server` and every other teardown reach it through that one method, standalone and warp-bare geometry alike.
A removal failure is logged at Debug and never fails the teardown.

Bound: reed removes only the socket file for the `-L` key it has just killed, only inside the teardown that killed it, only when the path is a socket and no server answers on it.
It never globs or sweeps the socket directory.
A server that restarts on the same key between the check and the removal is the residual race, accepted because the key is one reed has just torn down (D9).
This is the only production behavior change in the plan; the removal is a file operation, not a process spawn, so `PATTERN-spawn-observability` does not apply.

`socketfile_test.go` is untagged and spawns nothing; one table-driven test over a temp `TMUX_TMPDIR`-shaped directory covers: a stale socket (a `net.UnixListener` with `SetUnlinkOnClose(false)`, closed) is removed;
a socket with a live listener is kept;
a regular file at the path is kept;
a missing path returns no error.
The helper takes the socket directory as a parameter so the test needs no `t.Setenv` and calls `t.Parallel`.
In `contract_integration_test.go`, the `RemoveStrand` sole-strand case, whose teardown kills the server, asserts no socket file remains for its key afterwards.
`internal/reedengine/doc.go`'s teardown section states the removal and its bound.

**ImpactSummary:** one reedengine teardown method gains a bounded socket-file removal reached by `Down` and the force-reap path; no exported API changes.
