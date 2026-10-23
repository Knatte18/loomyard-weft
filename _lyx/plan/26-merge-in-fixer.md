# Card 26 — merge-in-fixer

**Create:**
- `plan:internal/landingshed#MergeIn.align` -> `func (m *MergeIn) align(ctx context.Context, tip, mergeCommit string, gateReason string) (shedengine.Outcome, shedengine.OutputPointer, error, bool)`
- `plan:internal/landingshed#TestMergeIn_Fixer` -> `func TestMergeIn_Fixer(t *testing.T)`

**Edit:**
- `internal/landingshed#MergeIn`
- `internal/landingshed#NewMergeIn`
- `internal/landingshed#MergeIn.Call`
- `internal/landingshed#MergeIn.writeRecord` -> `func (m *MergeIn) writeRecord(tip, mergeCommit, alignedTip string, merge mergeresolve.Result) error`
- `internal/landingshed#Deps`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#TestMergeIn_VerifyGate`
- `internal/landingshed#TestMergeIn_PlanVerifyFailureRecord`
- `internal/landingshed#TestMergeIn_MergeRecord`
- `internal/loomcli#landingDeps`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`

**Uses:**
- `plan:internal/landingshed#newFixerRun`
- `plan:internal/landingshed#FixCap`
- `plan:internal/landingshed#ReadFixerAttempts`
- `plan:internal/landingshed#RemoveFixerAttempts`
- `plan:internal/landingshed#FixerAttemptsPath`
- `plan:internal/landingshed#LandingStopPrefix`
- `plan:internal/landingshed#fakeFixerSeats`

**Intent:** Merge-In runs the merge fixer in `merge-in` mode, which aligns and fixes (D4).
After the merge, the conflict session and the post-merge clean check, `Call` runs the `landing` gate as today and then `align` instead of stopping on a failed gate.
`align` reads the entry's attempt record (`ReadFixerAttempts` under `Deps.RowEntry("Merge-In")`) and decides from three facts: whether the gate passed, whether the parent brought in commits since the record's aligned tip, and what the entry's record holds.
An absent aligned tip reads as the task's fork point from the parent, never as a merge base taken from the merged tip, so a run's first Merge-In over a parent that moved since the fork spawns the fixer; "brought in commits" is `HoldsWork(align-from tip, merged tip)` answering false.
The fork point is fixed once per entry: the first time `align` finds no aligned tip in the merge record and no `AlignFrom` in the entry's attempt record, it takes the merge base Merge-In read before this entry's first merge and writes it as the record's `AlignFrom` before deciding anything, and every resume of the entry reads that `AlignFrom` back, so a resumed Merge-In after an `escalate`, whose merge is up to date and whose fresh merge base would be the merged tip, still reads the parent commits the first merge brought in and runs the fixer again.
An attempt record that does not decode is a Stuck with Halt naming the file, with the way forward to remove the file by its path, which grants the entry a fresh cap, then `lyx loom resume`.
A merge that brought no parent commit over a passing gate spawns nothing, sets the aligned tip to the merged tip and is Done.
Parent commits brought in, whatever the gate says, or a failing gate, spawn card 11's fixer in `merge-in` mode on the attempts `landing.yaml`'s `merge_fix_attempts` leaves the entry; the first spawn writes the entry's fixer baseline, the HEAD after the merge and the conflict session committed, into the record before the turn, and every resume of the entry keeps that baseline.
A `fixed` verdict over a passing gate advances the aligned tip to the merged tip and is Done; Merge-In writes the aligned tip on every Done, and only Merge-In writes it.
An `escalate` verdict, a spent cap and the fixer's other failure exits halt with card 11's reasons; the aligned tip stays, so a resumed Merge-In runs the fixer again over the amended record, and a failed spawn is an error stop that spends no attempt.
A spent cap notes `spent` in the record; the resumed Merge-In of that entry, with no attempts left, reads the operator's fix as the alignment: over a passing gate it spawns no fixer, advances the aligned tip and is Done, and over a failing gate it halts again naming the log.

When the Code-Review perch's effective budget is 0, read through the new told `Deps.CodeReviewOff` (loom's `code_max_bounces`, or `review_max_bounces` when that is null, equal to 0; `landingDeps` fills it for loom from the loom config at call time, and it reads false for darn, which has no Merge-In), a `fixed` change is judged by `FixCap` over `LineChange(baseline, HEAD)` with `finalize_fix_max_lines` and the run's language matcher; a change over the caps halts with `LandingStopPrefix`, naming the fixer's commits (`baseline..HEAD`), with the way forward to review and fix on the task branch, then `lyx loom resume`, and notes `over-cap` in the record.
The resume after an over-cap halt writes the baseline afresh at the HEAD it resumes from and reads the operator's review as the alignment: over a passing gate it advances the aligned tip and is Done, and over a failing gate it runs the fixer on the attempts the entry has left.
Because the cap measures from the baseline, the parent's merged commits and the conflict resolution never count, and a budget above 0 sends a `fixed` change over a passing gate to the Code-Perch unjudged.
Every Done clears the entry's attempt record; a re-entry from another row, such as Merge-In after a Finalize reroute, finds a record naming another entry and starts at zero spent attempts.
A passing gate removes the landing failure record and a failing one writes it, as `verifyGate.check` does (D12).
Each halt is a Stuck with Halt, so the loom halt note is written (`PATTERN-friction-capture`) and the run stops at the row.

`writeRecord` takes the aligned tip it writes.
`TestMergeIn_Fixer` drives `Call` over the fixture with `fakeFixerSeats`; `newMergeInFixture` gains a `TaskHead` fake that advances when the merge and each fixer turn commit, a `HoldsWork` fake keyed by its two tips, and a `LineChange` fake that records the `from` and `to` it is called with.
It pins one row per acceptance case: no parent commit and a passing gate spawns nothing and advances the aligned tip; parent commits and a passing gate spawn the `merge-in` fixer, the first Merge-In included; a failing gate re-prompts until the gate passes or `merge_fix_attempts` is spent, then halts naming the log; `escalate` on a run's first Merge-In halts naming the report, and the resumed Merge-In, whose fake merge base now answers the merged tip, still reads the `AlignFrom` fork point and runs the fixer again; a resumed spent entry over a passing gate is Done with no spawn, and over a failing gate halts again naming the log; under `CodeReviewOff`, a fixer change past `finalize_fix_max_lines` or editing a `_test.go` file halts with `LandingStopPrefix` naming its commits, a merge whose parent changes and conflict resolution exceed the caps followed by a fixer change within them is Done, and in both rows the recorded `LineChange` `from` is the HEAD after the merge and the conflict session, never the pre-merge HEAD; a resume after an error stop calls `LineChange` from that same baseline; a resume after an over-cap halt writes the baseline afresh at the HEAD it resumes from and is Done over a passing gate, and over a failing gate runs the fixer on the attempts left; with a budget above 0 the same over-cap change with `fixed` over a passing gate is Done; a re-entry from another row starts at zero attempts while a resume of the same entry keeps its spent ones; an undecodable attempt record halts naming the file and its way forward.
The verify-gate and failure-record tests assert the fixer is spawned where they asserted a Stuck.

**ImpactSummary:** Merge-In's failed-gate Stuck becomes the fixer loop, and one told seam is added and filled by loom's landing assembly and the test kit.
