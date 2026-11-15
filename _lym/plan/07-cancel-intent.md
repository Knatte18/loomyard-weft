# Card 7 — cancel-intent

**Create:**
- `plan:internal/battencli#cancelledRunRefusal` -> `func cancelledRunRefusal(slug string) error`

**Edit:**
- `internal/battencli#cancelDeps`
- `internal/battencli#cancelReport`
- `internal/battencli#endRunPhase`
- `internal/battencli#clearRunPhase`
- `internal/battencli#cancelNotice`
- `internal/battencli#cancelEnvelope`
- `internal/battencli#battenCLI.cancelCmd`
- `internal/battencli#hubCancelDeps`
- `internal/battencli#childParentHolds`
- `internal/battencli#Command`
- `internal/battencli#battenCLI.checkRunStatus`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/battencli#DobbyTasks`
- `internal/battencli#cancelFake`
- `internal/battencli#cancelFake.deps`
- `internal/battencli#TestCancelRun`
- `internal/battencli#TestRunCmd_ResumeDispositions`
- `internal/battencli#TestBattenPreStep_RefusalKinds`
- `internal/battencli#TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord`
- `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`
- `internal/battencli#TestCancelIntegration_KeptPairJudgeOnly`
- `internal/battencli#TestCancelIntegration_RemoveTearsDownPairAndBranch`
- `internal/boardengine#RunLockedError.Error`
- `docs/overview.md`

**Uses:**
- `plan:internal/shedrun#WriteCancelIntent`
- `plan:internal/shedrun#FinishCancel`
- `plan:internal/pairteardown#ReadRun`
- `plan:internal/runfsm#Judge`
- `plan:internal/fabricengine#OriginParentHoldsTaskWork`
- `internal/battencli#battenCommitStatusDeps`
- `internal/boardengine/doc.go`

**Intent:** `lym batten cancel` records its intent and its finish in the batten record and keeps the record, per D3.

New `cancelDeps` seams, each filled by `hubCancelDeps`:
- `Verdict` returns the act rule's verdict for the slug: `pairteardown.ReadRun` then `runfsm.Judge` with `Cancel`, `Pause`, `Hold`, `ShedPause` and `PushOwed` set aside, so a paused, held or offline run is still cancelled (Shared Decisions).
- `WriteIntent(remove)` writes the cancel intent through `shedrun.WriteCancelIntent`, with the time now, `machinename.Name` and remove, reporting whether a record stood.
- `CommitRecord` commits the batten record with the run's other records and spawns today's push, through `battenCommitStatusDeps(location, slug)`.
  It refuses while `MergeActive` reports a merge on the prime's records side, naming the merge to conclude or abort there and the re-run, so no cancel ends a run with its intent uncommitted;
  `commitRunRecords` is not used here, since it skips a mid-merge commit silently.
- `FinishCancel(remove)` writes the finish through `shedrun.FinishCancel`, with the time now.

Order in `endRunPhase`:
1. After the reached-parent check, ask `Verdict`; any answer but act refuses with `Verdict.Refusal(slug, rerun)` before anything ends.
   With those inputs set aside it refuses a landed run and, once D4 fills it, a missing code tip.
   `judgeOrigin` already refuses a slug another machine owns, and the reached-parent check a batten `done` record, before the rule is asked,
   so the input that reaches this refusal is a slug whose board status is `done` with no batten record, read `Landed`.
   Its `TestCancelRun` row is that slug, refusing with the terminal wording; a hand-over row stays as an extra case.
2. A finished cancel that kept its pair, run again without `--remove`, refuses: `<slug> is already cancelled with its pair kept; way forward: run "lym batten cancel <slug> --remove"`.
   The input that reaches it is a status whose `Cancel.Finish.Remove` is false and remove false; a `TestCancelRun` row reaches it.
3. After the lock is taken and the Windows live-process check passes, and before `EndForCancel`, write the intent and commit it, so the intent is committed, and its push spawned, before anything ends.
   A failed write or commit refuses, naming the re-run, with nothing ended.
   With origin unreachable the commit is local and its push owed; the cancel ends the run's processes and strands and stops end-only, naming the re-run once origin answers, as today.
   A slug with no batten record writes no intent.

`clearRunPhase`:
- With `--remove`, it removes the pair and task branch as today, then deletes the batten record in the same step through `RemoveRecords`, whose commit is pushed;
  a slug released again then reads `queued`.
- Without it, it keeps the record: `FinishCancel(false)` sets the finish and the state `cancelled`, never `done`, and `CommitRecord` commits and pushes it.
  The dobby's done-run clear then never takes it, since that clear takes only `done`.
- The claim clear, the pause lifts and the notice stay as today.

`cancelReport` gains `RecordKept`, and `RecordsRemoved` is true only under `--remove`; `cancelEnvelope` adds `record_kept`.
`cancelCmd`'s Long text, the `batten` group's Long text in `Command`, `RunLockedError.Error` and `docs/overview.md`'s batten cancel paragraph say the record is kept with its finish, or deleted with the pair under `--remove`, and drop "removes the run's records".
`cancelCmd`'s end-only sentence says the cancel leaves its intent committed locally, where it says today that it leaves its marker.

`childParentHolds` reads the parent through `fabricengine.OriginParentHoldsTaskWork`, or `ParentHoldsTaskWork` under the push skip, so the cancel's reached-parent refusal and the dobby's reached-parent reading follow origin's fetched parent branch, never the local one (D1).

`cancelledRunRefusal` words the refusal of a start, step or resume over a cancelled record:
`<slug> was cancelled; way forward: run "lym batten cancel <slug> --remove", which removes its record and any pair, then <releaseCommandText(slug)> and "lym batten run <slug>"`.
`checkRunStatus` and `battenPreStepUnmarked` return it for `shedengine.StateCancelled` instead of the unrecognized-state error,
and `DobbyTasks` sets it as the start refusal of a released task whose record is cancelled.
Bound: a record a crash left after its pair's removal, with its task branch gone from origin, reads `queued` through the FSM as D1 says, and its start still refuses until `--remove` or `lym batten cleanup --apply` (card 9) deletes the record,
since seeding over a standing record would resume the cancelled run's row instead of a fresh start; `--remove` over such a leftover finds the record left and deletes it.

Tests:
- `cancelFake` gains fields for the four seams and `cancelFake.deps` defaults them to an act verdict and writes that succeed, each appending its name to `calls`, so existing rows run unchanged and the new rows assert order with `callsInOrder`.
- `TestCancelRun` (tier0) gains rows: the verdict refusal of a board `done` slug with no batten record, with the terminal wording, and a hand-over refusal; a batten-paused run cancelled (the shed pause set aside); the kept-finish re-run refusal; intent written and committed before `EndForCancel`, and none for a slug with no record;
  a failed intent commit, and a mid-merge commit, refusing with nothing ended; end-only with the intent committed locally; keep finishing with `cancelled` and no records removal; `--remove` deleting the record after the pair.
- `TestRunCmd_ResumeDispositions`, `TestBattenPreStep_RefusalKinds` and `TestDobbyTasks_ReadsTheStartRefusalOfAFinishedRunRecord` (tier0) gain a cancelled row asserting the refusal text.
- The three cancel integration tests (tier1) assert a plain cancel leaves the record with state `cancelled` and its finish committed on the records side, and `--remove` deletes it.
  `TestCancelIntegration_KeptPairJudgeOnly` also waits for the spawned records push and reads the record from origin's records branch with its finish (D3's on-origin criterion),
  and `TestCancelIntegration_BlockedRunWithNoPair` gains a row whose cancel stops after its intent push and whose record, read from origin's records branch, derives `cancelling` through `runfsm.Derive` (D3's other-machine criterion).

**ImpactSummary:** Changes the cancel's record handling and its envelope's keys, the run, step and dobby-start refusal for a cancelled record, the reached-parent read and four texts naming the old removal; all within battencli but one board error text and one doc.
