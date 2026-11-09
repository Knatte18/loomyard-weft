```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none (code-tests lens).

**Behaviour → pinning test**

1. The family filing turns the run's entry into a discussed project with kind, no status, cleared recipe and `depends_on`, kept title and labels, `order`, body (prefix, part list, former body), the whole record, and draft parts carrying their `depends_on`.
   - Pinned by `TestFileRaddle/AFamilyFilingTurnsTheRunEntryIntoTheProjectAndRepointsItsDependents`.
   - That test asserts neither `isolated`, `profile`, `run_settings`, `pull_request` nor `notify_on_land` is cleared.
   - `arrasProjectOf` is also pinned indirectly by `TestCheckRunLocks_FamilyProject`, which builds `now` with it.
2. A failed family write restores the entry's former record and removes the filed parts' record files.
   - Pinned by `TestFileRaddle/ARepointTheBoardRefusesFilesNothingAndRestoresTheEntrysRecord`, which asserts both the restored record and the absent `plan-a` record.
   - The branch where the entry held no record file before the write (the `fs.ErrNotExist` case in `convertRunEntry`) has no test.
3. The store write failing after the board write, then a resume, is pinned in `internal/loomshed/raddlefile_test.go`, but only against mocked seams.
   - The rows are `AFamilyFilingFoundAsItsOwnIsRepairedAndNotFiledTwice/WithTheStoreWritten` and `/WithTheStoreWriteLost`.
   - `HoldsRaddleFiling` on the family filing is asserted in the board test.
   - No test connects the two layers.
4. The run lock admission in `endsAsFamilyProject` is pinned by `TestCheckRunLocks_FamilyProject`.
   - Rows: derived project passes; changed title, changed `depends_on`, another producer, parts out of dependency order and an undiscussed part are refused.
   - Rows missing:
     - `now.Order` differing from the sequence in `after`.
     - A changed body or part list.
     - A part that is not draft, or not a task.
     - A part already in the loaded list, so it is not an added part.
     - No added parts (`len(parts) > 0`).
     - `now.Status` non-nil, and a changed discussed flag.
     - A Raddle-File producer in a state other than running.
   - The `order`-from-`after`-not-from-the-entry rule in the focus directive has no row.
5. Moving dependents onto the filed parts that no other part depends on:
   - Family outcome: pinned by the `waiter` assertion `[other plan-b plan-c]` in the family subtest. It covers sink selection, order and de-duplication.
   - One-task and `tasks` outcome (D10, "depends on the filed task"): none. No `FileRaddle` test files a non-family shape with a dependent on the run's entry.
   - Skipping done dependents: none.
6. A refused re-point:
   - Run-locked dependent: pinned by the `ARepoint…` subtest (`ErrRaddleRepointRefused`, `ErrRunLocked`, `"waiter"`).
   - Refusal from `validateWrite` (the `entry %q: %w` branch): none.
   - Halt naming the refusing entry, with no return to Raddle-Write: pinned by `ARefusedMoveOfTheDependenciesHaltsNamingTheEntry`.
7. Raddle-File writes the filing mark after the filing and never calls `EndEntry`: pinned by `AFamilyFilingMarksTheRunAndNeverEndsTheEntry`.
8. Raddle-File claims its own status first when another producer holds it: pinned by the same test, `UnderAnotherProducersStatus`.
9. The store-done and `AlreadyFiled` repair paths write the mark, then clear the status, skip the filing steps, and skip `EndEntry`: pinned by `AFamilyFilingFoundAsItsOwnIsRepairedAndNotFiledTwice`, both rows.
10. Halt over an entry that is already a project and fails both tests, naming `lym batten cancel <slug>` and the `order`, with nothing checked or filed: pinned by `AnEntryAlreadyAProjectNotFiledFromTheProposalHalts`.
11. Failure paths of `MarkFiled` and `ClearStatus` in `repairFamily` and after the filing: none.
12. `ClearProjectRunStatus`:
    - Clearing a run status on the project: pinned in the family subtest.
    - The no-op branches (absent entry, not a project, no run status): none.
13. A null status releases no member: only `SetStatus(plan, nil)` is exercised, and it runs on a status that is already nil.
    - The D8 criterion also names upsert, merge and batten cancel's claim clear. Those have no row in this diff.
14. Loom's status write skipping once the mark stands: the wrapper is pinned by `TestSkipAfterFamilyFiling`, whose rows are unmarked, marked and unreadable mark.
    - Not pinned: that `commitStatusDeps` applies the wrapper only to recipes with a Raddle-File row (the `recipeHasEngine` condition).
    - Not pinned: that `raddleFilingMarked` reads the mark from the prime under `ResolveRunID`.
15. Batten's status write skipping under the mark: pinned by `TestBattenCommitStatusDeps_SetBoardStatusWritesTheRunsEntry`.
16. The records commit staging the mark: pinned by `TestBattenRunCommitPaths`.
17. `ClearMotifRun` clearing a done arras run and removing the mark: pinned by `TestClearMotifRun_RemovesOnlyADoneMotifOrArrasRun/DoneArrasRunIsRemovedWithItsFilingMark`.
    - Restoring the mark on a failed removal: pinned by `…FinishesTheRemovalOfARunWhoseRecordsCommitFailed`.
    - Whether the row asserts the mark is absent after the removal is not visible in the diff. The row only asserts `wantCleared`.
18. The filing mark's path, relative path, round trip and idempotence: pinned by `TestFilingMark`.
19. The `Entry` seam now carrying `Kind`, `Order` and `StatusProducer`, and the real `ClaimStatus`, `MarkFiled` and `ClearStatus` wiring in `raddlewiring.go`: none. Only mocked seams exist, and `wire_seams_test.go` only lists them as intentionally nil for batten.
20. `RaddleFileProducer` matches the row name: pinned by `TheBoardsRaddleFileProducerIsTheRow`.

**Findings**

- **MEDIUM, design (D10)** — `internal/boardengine/raddlefile.go:143` (`repointDependents`) with `internal/boardengine/raddlefile_test.go`.
  - Summary: the one-task outcome's re-point of dependents onto the single filed task has no test.
  - Evidence: only the family subtest has a dependent on the run's entry, and it asserts the family result.
  - Fix: add a `tasks`-shape `FileRaddle` case with a dependent on the run's entry, plus a done dependent that stays unchanged.
- **MEDIUM, design (D6)** — `internal/loomcli/wiring.go:250` and `raddlewiring.go:212-278`.
  - Summary: no test shows that a Raddle-File recipe's real `commitStatusDeps` skips the board write under the mark, or that the real `Entry`, `ClaimStatus`, `MarkFiled` and `ClearStatus` wiring works.
  - Evidence: `TestSkipAfterFamilyFiling` tests only the wrapper.
  - Fix: add a test that builds `commitStatusDeps` for an arras recipe, writes the mark and asserts no board write, and for a non-Raddle-File recipe asserts the write still happens. Alternatively drive `raddleFileDeps` seams against a hub board fixture.
- **LOW, design** — `internal/boardengine/runlock.go:171` (`endsAsFamilyProject`) with `TestCheckRunLocks_FamilyProject`.
  - Summary: the table lacks rows for `order` or body differing from the `after` sequence, a non-draft part, a part already loaded, empty parts, a non-nil `now.Status`, and a Raddle-File status in a state other than running.
  - Evidence: the focus names `order` read from `after`, which no row pins.
  - Fix: add those rows to the table.
- **LOW, design** — `internal/boardengine/raddlefile.go:238` (the `validateWrite` branch of `repointDependents`).
  - Summary: the board-refusal branch other than a run lock is untested.
  - Fix: add a refusal row, for example a dependent whose rewritten `depends_on` the board rejects.
- **LOW, design** — `internal/boardengine/raddlefile.go:281` (`ClearProjectRunStatus`).
  - Summary: the no-op branches (absent, not a project, no run status) are untested.
  - Fix: add rows to the existing family subtest, or a short table.
- **LOW, design (D8)** — the family subtest.
  - Summary: the null-status-releases-no-member criterion for upsert, merge and batten cancel's claim clear is exercised only by `SetStatus(plan, nil)` on an already-nil status.
  - Fix: set a run status first so the nil write is a real change, or cite the existing test of those paths if one covers them.
- **LOW, design** — `internal/loomshed/raddlefile.go:445` and `:467`.
  - Summary: the failure of `MarkFiled` or `ClearStatus` (`p.fail` paths) is untested.
  - Fix: add rows using `f.failing["mark_filed"]` and `f.failing["clear_status"]`.
- **LOW, design** — `internal/boardengine/raddlefile.go:155`.
  - Summary: the `convertRunEntry` branch for an entry with no prior record file (removal instead of restore) is untested.
  - Fix: add a failed family filing over an entry with no stored record.
- **Test economy:** I found no redundant or duplicate tests and no new assertion on static prose.

**Tool calls:** none (my context already held every test and diff I needed).
