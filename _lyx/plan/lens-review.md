# Lens review

Every fork returned a report.

- criteria-coverage, BLOCKING, `dead` withholds confirmation though Decision 6 lets it confirm: fixed. A session is listed while its pane is live, and its state can read `dead` (`internal/shuttleengine/sessionstate.go:289`). Card 4 adds `dead` to the confirming set, with a row.
- criteria-coverage, MEDIUM, no row confirms over `idle-stalled`, `idle-fresh` or `asking`: fixed with a card 4 row that also covers `dead`.
- criteria-coverage and row-reachability, MEDIUM, the pre-call re-check is never reached: fixed. Card 3's `fakeWorld` gains an `onStatus` hook, and card 4 adds two sub-rows, a resume and a cancel after the confirmed reading.
- criteria-coverage and row-reachability, MEDIUM, no row reaches `ResumedDuringGrace` or a `Pause` error: fixed. The fake `Pause` can now produce both, and card 4 adds rows.
- row-reachability, MEDIUM, the resume-during-confirming rows confirm before any `onSleep`: fixed. Both rows now seed a busy first reading.
- row-reachability, MEDIUM, the `TestCancel` kill-seam row has no hook: fixed with an `onKill` hook on `fakeWorld`.
- row-reachability, LOW, the racing `TestSubmit` row has no mutex: fixed; `fakeWorld` gains a mutex.
- row-reachability, LOW, the `Added`-error row's "no Esc" would hold vacuously: fixed; the row seeds a busy session.
- row-reachability, LOW, there are no rows for a spawn error, a dead-daemon lift error or a reading error: fixed with rows in cards 3 and 4.
- row-reachability, LOW, the CLI integration test refuses off Linux: fixed with the build constraint `integration && linux`.
- criteria-coverage, LOW, no `Pause` error row: fixed, as above.
- criteria-coverage, LOW, the trigger backoff is not asserted: fixed. `fakeWorld` records the sleep durations, and the `--after` row asserts 1s, 2s, 4s, 8s, then back to 1s.
- criteria-coverage, LOW, `Worktree-Teardown` already reached on the first reading: fixed with a sub-row.
- criteria-coverage, LOW, the orch and an external caller are allowed on `cancel` and `watch`: fixed. The `TestCancel` row runs both callers, and the `TestWatch` rows use the orch and an external caller.
- criteria-coverage, LOW, the refusal-spec section and the denied forms lie outside the Scope list: rejected. Both are edits forced by the repo's own contracts. The `## pause` section of `contracts/specs/refusal-spec.md` sets the precedent, and the list in `cmd/lyx/norecoverydeny_test.go:24` "follows" claudeengine's deny table.
- forced-edits, BLOCKING, `internal/shortcutcli#TestMount_File` pins the shipped map exactly: fixed. Card 7 lists the test and adds `p` and `r` (`internal/shortcutcli/shortcut_test.go:212-220`).
- forced-edits, LOW, the header of `hookcmd_integration_test.go` is stale: fixed; card 6 lists the file.
- forced-edits, LOW, the doc comment of `TestBuildSettings_AgentAndBashHooks` names only the fork guard: fixed in card 6's Intent.
- forced-edits, LOW, no import-allowlist test for `hibernateengine`: rejected. `tenderengine`, the engine it mirrors, carries none either, and the told-geometry rule there is kept by review.
- test-economy, LOW, the two lift-nothing rows duplicate one predicate: fixed; they are folded into one looping row.
