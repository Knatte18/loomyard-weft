# Card 11 — reedengine-untag

**Edit:**
- `internal/reedengine#`

**Intent:**
Run the untag procedure (Shared Decisions, `untag-procedure`) on `internal/reedengine`'s `tmux` files (`tmux`, `tmux && linux` and `tmux && !windows`).

Most of them pin the contract with a real tmux server (`contract_integration_test.go`, `attachgeometry_integration_test.go`, `watchdog_integration_test.go`) and stay `tmux`.
A test whose assertion is about reed's own state, records, layout plan or launch script, and which reaches tmux only through the engine's multiplexer dependency, moves to an untagged file over a fake multiplexer; when the engine takes that dependency only as a package-level value, add it as a struct field or option that production sets to today's value (D6).
A fake multiplexer used by this package alone stays in its `_test.go` files.

Then merge scenario families (Shared Decisions, `scenario-merge`) among the tests that stay `tmux`: tests that each start their own tmux server through `tmuxkit` and differ only in the steps after it.
A test moved to the untagged tier above never joins a `tmux` scenario.
The commit message names each merged family and how many fixture builds it removed.

`TestMain` keeps `tmuxkit.Main` while any tagged file remains.
`spawnwatchdog_test.go` and `panebin_test.go` keep their `os.Executable` guard rules: no test may call `SpawnWatchdog` or re-exec the test binary.

**ImpactSummary:** Test files of one package, plus a seam field in its production code only where a moved test needs one; production passes today's value, so reed's callers see no change.

**Verify:**
go test -tags tmux -count=1 ./internal/reedengine/
go vet -tags integration ./internal/reedengine/
go vet -tags llm ./internal/reedengine/
