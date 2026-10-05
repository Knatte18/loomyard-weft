# Card 18 — Move ad hoc tmux keys in tests onto tmuxkit

**Edit:**
- `internal/reedengine/contract_integration_test.go`
- `internal/reedcli/smoke_attach_test.go`
- `internal/reedcli/smoke_dotfill_test.go`
- `internal/reedcli/smoke_dotfill_measure_test.go`
- `internal/reedcli/smoke_test.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/reedcli/smoke_coldstart_test.go`
- `internal/reedcli/smoke_debuglog_test.go`
- `internal/reedcli/smoke_resume_test.go`
- `internal/reedcli/smoke_teardown_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/mouse_boot_integration_test.go`
- `internal/standalonegeom/naming_integration_test.go`

**Uses:**
- `internal/testkit/tmuxkit#KillOnCleanup`

**Intent:**
Every test that starts a tmux server takes its key from `tmuxkit` (D9), so its socket file is removed after `kill-server`.
The reedengine contract fixtures (keys from `ServerName(hub)` torn down by a raw `e.tmux.run("kill-server")` in `t.Cleanup`) register their key through `tmuxkit.KillOnCleanup` instead, keeping `e.Down()` ahead of it.
The reedcli attach and dotfill harnesses, which start their own harness server on a hand-formed `-L` key, take it from `tmuxkit.Socket`, and the reed server key they drive goes through `KillOnCleanup`.
`internal/reedcli/smoke_test.go`'s shared fixture helpers that start or tear down a reed server register the hub's key the same way.

The other files listed above hold a raw `kill-server` on a key the test did not mint (`standalonegeom`'s `"lyx-"+hash8` key, the reedcli naming, cold-start, debug-log, resume and teardown tests, the reedengine ensure-session and mouse-boot tests), found by grepping the tests for `kill-server`; each moves to the same helper.
Fix every further source card 17's survey named that this list misses in this card too: a test that leaves a socket file registers its key through `KillOnCleanup`, and a test that builds a subprocess environment from scratch (an `exec.Cmd.Env` not derived from `os.Environ()`) passes `TMUX_TMPDIR` through so the child's sockets land under `tmuxkit.Main`'s directory.
Add each such file to this card's commit.
Then re-run the survey: a full `go test -tags integration,tmux -count=1 ./...` adds no file to `/tmp/tmux-$UID/` (acceptance).
A test killed by a timeout or panic skips cleanup and can still leave one; the survey is run on a passing suite.

**ImpactSummary:** test-only, in reedengine and reedcli fixtures; teardown order is kept and only the socket-file removal is added.
