# Fixer-report, code review round 1

Every finding is fixed.
Tier 0 and tier 1 of `lym gate test` pass over `internal/fabricengine`, `internal/fabriccli`, `internal/pauserecord`, `internal/pairteardown` and `internal/runproc`.
Findings share files, so the commits are grouped by theme and name their finding ids, not one commit per finding.

## Gate repairs

Two gate findings followed the first report, each repaired in its own commit after the five commits below.

- The `remove.go` header comment of F9 was broken mid-sentence at a comma, so no test ran.
  It is now two sentences, one per line (`internal/fabricengine/remove.go:2-3`), with no change to its meaning.
- `cmd/lym`'s `TestTierPurity_UntaggedTestsSpawnNothing` rejected `exec.Command` in the tier 0 file `remoteteardown_test.go`.
  `TestChildHold_Finish`, `TestChildHold_Finish_CarriesThePauseExemption` and `recordingStdin`, which start a stand-in child process, moved to the tier 1 file `remoteteardown_childhold_integration_test.go` with `tiergate.Require(t, 1)`.
  Their rows are unchanged, so the F2 and F4 coverage described below now runs at tier 1.

## Changes

- **F1** (fixed as proposed, with one choice the `fix` key left open):
  `deleteTaskBranchOnOrigin` and `deleteTaskBranchAtTip` (`remove.go`) return a third value, `failed`, set for a failed lookup, fetch or delete and left false for a gate refusal or a rejected lease (`gitrepo.ErrLeaseRejected`).
  `remoteOutcome.taskFailed` carries it.
  `noticeRemoteTeardown` now builds the notice from `failedRemoteSteps`, which lists the archive failure, the weft delete failure and the task delete failure, so a second failure is no longer dropped.
  The kept reason keeps the failure text, so `RemoteCodeBranchKeptReason` is unchanged for in-process callers.
  Commit 26dc54b8.
- **F2** (fixed as proposed):
  `remoteteardown_integration_test.go` runs `RunTeardownChild` on a hubforge hub whose origin lacks the weft tip and records the order through `archiveTagPushHook` and `removeBeforeWeftDeleteHook`: the branch tip is on origin at the tag push, the tag is on origin at the delete, and a read lock on the teardown lock fails at both steps and succeeds after the return.
  A second row fails the archive tag and asserts the delete never ran, the pushed branch stayed, one notice was mailed and the lock was released.
  `TestChildHold_Finish` covers the nil-job path (input closed, child waited for), the handed-off outcome and an unwritable job.
  `removefields_test.go` gets a `remote_handed_off` row and the key joins the absent-when-empty list.
  The test hub deletes the sibling worktree and local weft branch first, as a removal does before the handover, since the origin delete is refused while the branch is checked out.
  Commit a8855c26 and, for `TestChildHold_Finish`, 26dc54b8.
- **F3** (fixed as proposed):
  `TestNoticeRemoteTeardown` in `remoteteardown_test.go` installs a recording notifier and covers an archive failure with and without `WeftTip` for a removal and a cleanup, a discard failure with and without `TaskTip`, a weft delete failure, a task delete failure, both deletes failing in one notice, a kept task branch, a kept weft branch, a clean outcome, and no notifier installed.
  Commit 26dc54b8.
- **F4** (fixed differently):
  The proposed change, `pauserecord.SetGateExemptAll` in `teardownChildCmd`, is not made.
  Only the removal exempts its own process (`fabric.go:1103`); the cleanup sweep and the discard start the same child and are not exempt unless their caller is, so an unconditional exemption would widen their network access under a pause.
  The job now carries `ExemptPause`, read from the writing process's gate through the new `pauserecord.GateExemptsAll`, and `RunTeardownChild` runs exempt only when it is set.
  `fabriccli/doc.go` states that the exemption travels in the job and that the respawned push children keep their own gates.
  `teardownchild.go` also logs a Warn naming the way forward when a signal ends the child.
  Rows: `TestChildHold_Finish_CarriesThePauseExemption` and a `GateExemptsAll` subtest in `hold_test.go`.
  Commit fb2abde4, with the job field and the carrying code in 26dc54b8.
- **F5** (fixed differently):
  The child registers under a new role, `runproc.RoleTeardown`, and `endOwnedProcesses` (`pairteardown/teardown.go`) always spares that role, which covers `cancel.go`, `skip.go` and `teardown.go`.
  The tender end verbs stay unsparing: `teardownLockRefusal` names them as the operator's deliberate end.
  The proposal's second half, a Warn when a signal ends the child, is the F4 signal watch.
  `pairteardown/doc.go` and the `endOwned` field comment say the child is spared.
  Commit 2333dd08.
- **F6** (fixed as proposed, one condition made textual):
  `rerunWayForward` picks the cleanup sweep for a cleanup job and the removal otherwise.
  A removal's archive failure with a weft tip names the tag push, then the re-run "when origin still holds a branch of the pair", since the job cannot know that without a network read.
  A cleanup names the tag push, then the sweep.
  Commit 26dc54b8.
- **F7** (fixed as proposed):
  `RunTeardownChild` calls `noticeUnreadRemoteTeardown` when `io.ReadAll` or `json.Unmarshal` fails, and returns the error.
  `TestRunTeardownChild_NoticesAnUnreadableJob` pins the single notice.
  Commit 26dc54b8.
- **F8** (fixed as proposed):
  The `RemoveStepArchive` comment, the step-set comment, the `Steps` and `ArchiveTag` field docs, and `CleanupBranchEntry.ArchiveTag` describe the local tag whose push is the child's.
  Commit 26dc54b8.
- **F9** (fixed as proposed):
  `PATTERN-fabric-git.md` lines 30-35 and the `remove.go` header describe the local tag, the local deletes, then the child's tag push and origin delete.
  `docs/overview.md`'s teardown sentence follows.
  Commits 26dc54b8 and ec9aec14.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `pairteardown/doc.go` and the `endOwned` comment omit the always-spared teardown child: fixed, 2333dd08.
- fix-consistency and fix-affected-set, LOW, `docs/overview.md:740` states the old archive order: fixed, ec9aec14.
- fix-consistency, LOW, `anchor.go:411` and `anchordeps.go:447` spare lists lack the teardown role: disputed, premise false (see Disputed).
- fix-consistency, LOW, `fabricengine/doc.go` omits that every failed step and an unreadable job are mailed: fixed, 26dc54b8.
- fix-affected-set, LOW, `cleanup.go:101` documents `ArchiveTag` as pushed: fixed, 26dc54b8.
- fix-affected-set and fix-proposal, MEDIUM, F4's unconditional exemption widens cleanup and discard: fixed with an own change, the job carries the exemption (fb2abde4).
- fix-proposal, MEDIUM, F6's tag-only way forward drops the `--remote` deletes: fixed, the way forward names the re-run when origin still holds a branch (26dc54b8).
- fix-proposal, LOW, `warnWhenEnded` skips deferred releases and names the removal as the way forward for a discard child: fixed for the way forward text, which now names the removal, the cleanup sweep and the discard (fb2abde4); the skipped defers are harmless, since the OS releases the lock file at exit, and the signal path's missing test is deferred (see Deferred).
- fix-proposal, LOW, anchor spare lists (same item as the fix-consistency one): disputed, premise false.
- fix-affected-set left unchecked: whether `RunTeardownChild` has rows for the cleanup and discard kinds, which the review's F2 fix did not ask for; the integration rows drive the removal kind only.

All four forks returned a report.

## Disputed

- `premise false`, the anchor spare lists (fix-consistency 3, fix-proposal 3):
  `anchordeps.go:240` ends `runproc.EndSparing(runProcDir, dobbyOwner(worktree), ...)`, the dobby owner of a worktree, while the teardown child registers under the pair's slug (`remoteteardown.go`, `spawnTeardownHold`).
  The move's end never reaches the slug's records, so the push and sync spare list needs no teardown role, and `anchor_test.go:506` pins that list.

## Deferred

- A test of `warnWhenEnded`'s signal path (fix-proposal 4): it needs a process the test signals, and the package has no cross-platform helper for that; the Warn text is fixed.
- `docs/overview.md:741` still says a failed archive push halts the batten row: with the push in the detached child that is no longer how a push failure surfaces, and what the row does on a handed-over removal is batten's contract, outside this review's findings, so it is left for the operator to settle.
