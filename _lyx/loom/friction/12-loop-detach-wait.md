# Batch 12: loop-detach-wait, watchdog, idle key, loom arming, status strand

Card 12's Create list named `testmain_integration_test.go` but not the untagged `testmain_test.go` that `PATTERN-test-isolation` demands once a package has an `integration` test file, so the first gate run failed `TestTmuxIsolation_TaggedPackagesRunThroughTmuxkitMain` until I added it.
The card also gave `waitLoopEnvelope(ctx, spec, spawned)` no parameter for the followed loop's id, which the waiter must adopt and change during the wait, so the signature carries a `*loopFollow` instead.

Card 16 listed only `smoke_driverstrand_test.go`, but `smoke_drivingsurface_test.go` and `smoke_starttail_test.go` (both `tmux` tier) also asserted that an llm-driven start removes the status strand, so they changed too.
`TestSpawnObservability_ProductionSpawnsAreLogged` stays red from batch 01: `internal/proc/tied.go` calls `exec.CommandContext` only to read the default `Cancel`, and trips the file-level logger-import scan.
