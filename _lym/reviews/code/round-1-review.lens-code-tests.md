```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: for each behaviour the diff adds or changes, the test that pins it, or none.**

I had to use test-name greps and a few diff excerpts rather than every test body. Rows I could not see are marked "not confirmed".

**Pinned**

| Behaviour | Test that pins it |
|---|---|
| Hold written, owner entry turned into the hold, operator entry kept | `holdEntries` test in `holds_test.go:90`; `adopt_test.go` rows `OwnerEntryBecomesTheHoldWithItsPauseTimeKept`, `AnotherCallersEntryIsKeptAndItsSlugIsNotHeld` |
| Hold before the owner write; failed or lost owner write leaves the holds | `RefusedOwnerWriteLeavesTheHoldsAndListsThem`, `LostOwnerWriteLeavesTheHoldAndNamesTheRerun` |
| Live pair, owned-elsewhere run and front task not held; released task held | `KeepLocalOverALivePairLeavesTheRunUnheld…`, `ReleasedTaskOfAnAdoptedBeamIsHeldAndAFrontTaskIsNot` |
| Re-run never holds the fix task; hand adopt resets the tries | `SecondAdoptRightAfter…`, `ReRunWithHoldsStandingAndNoOwnerChangeStartsTheCheckAndResetsTheTries` |
| Green end: lift only standing holds, other subject's and meanwhile-appended holds kept, hand-lifted skipped, failed step reported while the others go on, first passes, Finalize-halted resume, fix-task removal, tries reset | `TestGreenEnd` (`green_test.go:186–374`) |
| Fix loop: slug scheme, next-N from the board, run records and branches, try count, removal before filing, no-file cases, gate label gated by vocabulary | `TestFailedEnd` (including the "vocabulary without gate" row at `:185`), `TestFixSlug`, `TestFixBody` |
| Front-task file, remove and settle (closed table, owner-only, nothing-left counted as done, record lock) | `TestFile`, `TestRemove`, `TestSettle`; `hub_integration_test.go` |
| `front` and `on_land` refused when changed and dropped when equal on upsert, upsert-batch and merge; project, kind and rename refused | `stepFrontTask` in `boardcli/verbs_test.go`; `TestFrontTask` |
| Dobby front start past `max_runs`, front-first view, settle seam, held run gets no pass or lift pass | `TestStartReady_FrontTask`, `owned_test.go:132`, `TestPlanPass`; `HoldCallerEntryReadsHeld…` in `beamcli/dobbypairanchor_test.go:133` |
| Darn second stop marker, stale-marker removal, unwired seam | `TestDarnWrite_Call` rows |
| `adopt_fix_tries` default, override and a negative value refused | `config_test.go` rows |
| `pauserecord.Coverage.Held` | `TestCoverageOf` (not confirmed, since the diff adds 20 lines there) |
| `TaskBranchSlugs` | `TestTaskBranchSlugs` |

**Findings**

1. **MEDIUM, design.** `internal/fronttask/hub.go:114` (`startAction`) has no test. It returns an error under `go test`, so the argument vector, the detached spawn, the logged spawn that `PATTERN-spawn-observability` requires, and the output file are unpinned. `TestSettle` reaches it only through a fake `RunAction`.
   - Evidence: `hub.go:115–117`.
   - Fix: add a seam for the executable or a spawn function, then assert `adopt check &lt;subject&gt;` and the logged line. Alternatively, test the `actions` row's `args` output directly.

2. **LOW, design.** `fronttask.HubSettleDeps` (`hub.go:74–110`), the owner read for the subject, has no test.
   - `hub_integration_test.go` has one test, `TestHubDeps_FilesAndRemovesOnTheHubBoard`, named for `HubDeps`.
   - The "on the subject's owner only" rule is pinned only through the `OwnedHere` fake in `TestSettle`.
   - Evidence: `hub.go:87–104`.

3. **LOW, design (not confirmed).** The dobby wiring in `internal/beamcli/dobbydeps.go` (+6 lines) has no test I could see. That covers `SettleFronts: fronttask.Settle` and the `Project` passed to `ViewInput`.
   - The only `beamcli` test I confirmed is the `Held` read.
   - Without a wiring test, a nil `SettleFronts` would silently disable the land action.

4. **LOW, design (not confirmed).** `boardengine/replay.go` (+2) and `boardengine/task.go` (+4) carry the `front` and `on_land` fields through replay and the task type. I saw no replay or round-trip row for them.
   - `front_test.go` and `stepFrontTask` check the stored fields, but not a Bolt replay on another machine.
   - Under `PATTERN-tolerant-records` and machine switching, that replay is the path that matters.

5. **LOW, design.** `hubFixDeps` (`fixloop.go:~320`) reads `TryLimit` from `dobbyengine.LoadConfig(ConfigDirs(worktree))`, and the board vocabulary from `board.Vocabulary().Known`. No test covers that wiring.
   - `config_test.go` covers `LoadConfig` itself, and `TestFailedEnd` covers the logic with fakes.
   - The `Known` row therefore never runs against a real board template. The finding the vocabulary-gated label fixes was about exactly that case.

No missing-test finding for the holds file's lock behaviour: the removal under one update is exercised by the "holds appended meanwhile" row. I saw no concurrent-writer test for `appendHolds`, which I consider acceptable.

**Tool calls (2):**
1. Grepped test names and rows across the new and changed test files, because my context held no test bodies.
2. Read the diffs of `boardcli/verbs_test.go`, `boardengine/front_test.go`, `dobbyengine/config_test.go` and `loomshed/darnwrite_test.go`, plus the adopt and starts test rows and a grep for `ResetTries` and `writeTries` callers, for the same reason.
