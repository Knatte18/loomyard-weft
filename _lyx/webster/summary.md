# Logging cleanup: logger episodes and level policy applied across lyx

All four batches (cards 1–19) recorded `done`, batch head `50a8f694153bfce29c87a0f0c6950d79088482e6`.

## Batch 01 (cards 1–2)

The logger's stderr half defaults to `io.Discard` under `go test`, and test restores return to that default.
`logger.Episode` and `EpisodeSet` were added with the three new Level policy rules and `PATTERN-log-levels`.
No deviations.

## Batch 03 (cards 3–8)

Reed's resize re-apply failure, watch-tick and name-repair callee failures log once per episode.
The watchdog retires and respawns on a replaced binary, and lock contention exits at Debug.
Session-name read failures log at Debug for a missing transcript, else Warn once per session.
The create-worktree line dropped its duplicate mutations field.
Batten waits log Info once per state with a dedupe key, failed steps Warn.
Deviations: `internal/reedengine/generation.go` and `internal/reedengine/watchloop_test.go`.
The fork replaced card 8's single `reasonKey` with a `reasonKeys` map, because the halted arm's alternating reasons made one key log twice per check; a friction note records it.

## Batch 09 (cards 9–13)

Battenshed's polled seam failures log once per episode, and repeated goto, resume and spawn attempts log Info once.
The idle probe and busy send error carry a draft kind.
Shuttle's polled failures log once per episode, and a held re-prompt logs Info once.
Orch probe states agree, draft holds carry a constant reason, and tick failures log once.
The loop watchdog's and waiter's read failures log once per episode.
Deviations: `internal/battenshed/innerrun.go`, `innerrun_failures_test.go`, `internal/orchengine/watcher_test.go`, `watchloop_test.go`, `internal/shuttleengine/activity.go`, `fakes_test.go`, `run_test.go`, `wait_test.go`.
Three test rows were adapted to the code (cards 10, 11, 12), recorded in a friction note.

## Batch 14 (cards 14–19)

The loop's idle-release hook and the parent-lock wait's pause-flag read log once per episode, and a release logs Info.
`fabricengine.LogArchiveDir` names the hub log archive root.
Pair teardown copies the pair's trace and verify logs into it and prunes to 30 days.
Every landing queues a landed notice naming the tasks it made ready.
The orch stencil routes every landed notice to After landing and catches a dropped one at its next wake.
Deviations: `internal/landingshed/parentguard_test.go`, `internal/loomcli/looplock.go`, `looplock_test.go`, `internal/pairteardown/archive.go`, `archive_test.go`.
Card 19's four claim rows became six, as a row holds one phrase.

## Verify gate fixes

The gate failed on two `internal/gatecli` integration scenarios: `an_inherited_held_slot_runs_without_acquiring` and `a_target_outside_every_hub_runs_unslotted_and_logs_it`.
Both expected log lines on stderr and got none, because card 1 made the logger discard stderr under `go test` and the gate test helper stands in for `lyx`'s main.
The fixer's `fix:` commit `a030f2424` makes `runGateHelper` in `internal/gatecli/test_integration_test.go` call `logger.SetOutput(os.Stderr)` before raising verbosity; no assertions changed.
The fixer ran `lyx gate test --tags integration ./internal/gatecli`, which passed; it did not check other packages for helper binaries that read logger output from stderr.
