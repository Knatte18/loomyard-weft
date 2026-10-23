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
The fork point is fixed once per entry, before the merge: right after `readBefore`, `Call` reads the entry's attempt record and, when the merge record holds no aligned tip and the attempt record no `AlignFrom`, reads the task's fork point, `MergeBase(ParentBranch)` before the merge, and writes it as the record's `AlignFrom`; every later `Call` and `align` of the entry read that `AlignFrom` back, so a resumed Merge-In after an `escalate`, or after an error stop anywhere between the merge and `align`, whose merge is up to date and whose fresh merge base would be the merged tip, still reads the parent commits the first merge brought in and runs the fixer.
An attempt record that does not decode is a Stuck with Halt whose reason opens with `LandingStopPrefix`, naming the file, with the way forward to remove the file by its path, which grants the entry a fresh cap, then `lyx loom resume`; the prefix hands it to the orch, so no agent driver lifts it (D11).
A merge that brought no parent commit over a passing gate spawns nothing, sets the aligned tip to the merged tip and is Done.
Parent commits brought in, whatever the gate says, or a failing gate, spawn card 11's fixer in `merge-in` mode on the attempts `landing.yaml`'s `merge_fix_attempts` leaves the entry; the first spawn writes the entry's fixer baseline, the HEAD after the merge and the conflict session committed, into the record before the turn, and every resume of the entry but the one after an over-cap halt keeps that baseline.
`align` passes the fixer the aligned tip, or `AlignFrom` when the merge record holds none, as the from tip, and the merged tip.
A `fixed` verdict over a passing gate advances the aligned tip to the merged tip and is Done; Merge-In writes the aligned tip on every Done, and only Merge-In writes it.
An `escalate` verdict, a spent cap and the fixer's other failure exits halt with card 11's reasons; the aligned tip stays, so a resumed Merge-In runs the fixer again over the amended record, and a failed spawn is an error stop that spends no attempt.
A spent cap notes `spent` in the record, and every halt other than a spent cap and an over-cap change leaves `Halt` empty, so `Halt` names only the halt that last stopped the entry (card 10).
An entry has no attempts left when its `Spent` reaches `merge_fix_attempts`, whatever the `Halt` note says, so a `timeout`, `no-report`, `dirty` or `pushed` end on the last turn leaves it as spent as a spent cap does.
The resumed Merge-In of an entry whose record notes no `over-cap` and that has no attempts left reads the operator's fix as the alignment and spawns no fixer: over a passing gate it judges the entry's change as a `fixed` change is judged below, so under `CodeReviewOff` a change from the baseline over the caps halts over-cap and one within them advances the aligned tip and is Done, and over a failing gate it halts again with `LandingStopPrefix`, naming the log.

When the Code-Review perch's effective budget is 0, read through the new told `Deps.CodeReviewOff` (loom's `code_max_bounces`, or `review_max_bounces` when that is null, equal to 0; `landingDeps` fills it for loom from the loom config at call time, and it reads false for darn, which has no Merge-In; a nil `CodeReviewOff` reads as the budget being 0, so the caps apply and a fixer change is judged rather than let through, and a failed config read is an error stop naming the config), a `fixed` change is judged by `FixCap` over `LineChange(baseline, HEAD)` with `finalize_fix_max_lines` and the run's language matcher; a change over the caps halts with `LandingStopPrefix`, naming the fixer's commits (`baseline..HEAD`), with the way forward to review and fix on the task branch, then `lyx loom resume`, and notes `over-cap` in the record.
The resume after an over-cap halt writes the baseline afresh at the HEAD it resumes from, clearing `Halt` in the same record write, and reads the operator's review as the alignment: over a passing gate it advances the aligned tip and is Done, and over a failing gate it runs the fixer on the attempts the entry has left, or halts again with `LandingStopPrefix`, naming the log, when none are left.
Because the cap measures from the baseline, the parent's merged commits and the conflict resolution never count, and a budget above 0 sends a `fixed` change over a passing gate to the Code-Perch unjudged.
Every Done clears the entry's attempt record; a re-entry from another row, such as Merge-In after a Finalize reroute, finds a record naming another entry and starts at zero spent attempts.
A passing gate removes the landing failure record and a failing one writes it, as `verifyGate.check` does (D12).
Each halt is a Stuck with Halt, so the loom halt note is written (`PATTERN-friction-capture`) and the run stops at the row.

`writeRecord` takes the aligned tip it writes.
`TestMergeIn_Fixer` drives `Call` over the fixture with `fakeFixerSeats`; `newMergeInFixture` gains a `TaskHead` fake that advances when the merge and each fixer turn commit, a `HoldsWork` fake keyed by its two tips, and a `LineChange` fake that records the `from` and `to` it is called with.
It pins one row per acceptance case: no parent commit and a passing gate spawns nothing and advances the aligned tip; parent commits and a passing gate spawn the `merge-in` fixer, the first Merge-In included, whose spawned table carries the fork point as its aligned tip; a failing gate re-prompts until the gate passes or `merge_fix_attempts` is spent, then halts naming the log; `escalate` on a run's first Merge-In halts naming the report, and the resumed Merge-In, whose fake merge base now answers the merged tip, still reads the `AlignFrom` fork point and runs the fixer again; a run's first Merge-In that stops with an error after the merge, at a `landing` gate infrastructure error, and is resumed with its merge now up to date still spawns the `merge-in` fixer; a resumed spent entry over a passing gate is Done with no spawn, and over a failing gate halts again naming the log; an entry whose last turn ended as the timeout end is resumed with no spawn, Done over a passing gate, halting over-cap under `CodeReviewOff` when that last turn's commits put the change past the caps, and halting naming the log over a failing gate; a nil `CodeReviewOff` applies the caps; under `CodeReviewOff`, a fixer change past `finalize_fix_max_lines` or editing a `_test.go` file halts with `LandingStopPrefix` naming its commits, a merge whose parent changes and conflict resolution exceed the caps followed by a fixer change within them is Done, and in both rows the recorded `LineChange` `from` is the HEAD after the merge and the conflict session, never the pre-merge HEAD; a resume after an error stop calls `LineChange` from that same baseline; a resume after an over-cap halt writes the baseline afresh at the HEAD it resumes from, clears `Halt`, and is Done over a passing gate, and over a failing gate runs the fixer on the attempts left; under `CodeReviewOff`, an over-cap halt resumed over a failing gate whose fixer turn commits a change and escalates is resumed again and calls `LineChange` from the baseline written at the first resume, never from the HEAD the second resume starts from, so the escalated turn's commits still count; with a budget above 0 the same over-cap change with `fixed` over a passing gate is Done; a re-entry from another row starts at zero attempts while a resume of the same entry keeps its spent ones; an undecodable attempt record halts with `LandingStopPrefix`, naming the file and its way forward.
The verify-gate and failure-record tests assert the fixer is spawned where they asserted a Stuck.

**ImpactSummary:** Merge-In's failed-gate Stuck becomes the fixer loop, and one told seam is added and filled by loom's landing assembly and the test kit.
