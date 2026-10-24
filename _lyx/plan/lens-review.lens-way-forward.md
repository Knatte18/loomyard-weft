```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. For each refusal, stop or reset the cards add, I checked whether the next step it names can succeed from the state it leaves. Six findings.

1. **MEDIUM — Card 30 (beam-describe-producer), D11.** The Beam-Describe refusal of an empty description names a next step the run lock refuses.
   - The card says a finding names "the board entry's title or brief to fix, then `lyx batten run <slug>`".
   - The producer runs inside the beam's landing, after batten's claim made the project entry run-held.
   - `internal/boardengine#checkRunLocks` and `internal/boardengine#scopeChanged` refuse a Title or Brief change on a run-held entry.
   - So the named edit cannot succeed. The only exit is cancelling the landing, which the refusal does not name.
   - Reached by an empty brief with no commit subjects.

2. **MEDIUM — Card 34 (beam-create), D9 step 2.** A nested beam whose parent beam's pair is absent on this machine stops with a wrong way forward.
   - `Add` is called with `Parent` from `ParentBeam`.
   - Card 4's `resolveAddParent` refuses that parent as "not a pair of this hub" and names `lyx fabric pairs`. Listing pairs does not create the parent.
   - Card 28's `beamForkPrecondition` names `lyx beam create <beam>` or `lyx adopt <beam>` for this state, but card 34 applies no such precondition.

3. **MEDIUM — Card 34 (beam-create), D9 step 2 and the D6 interaction.** Step 2 is "skipped when the pair exists", and no step marks a pair that exists without the mark.
   - Card 4 has `Add` refuse `ErrBranchExists` when the beam's local branch is left from a removed pair.
   - Card 5 makes that refusal name `lyx fabric add <slug> --adopt-local`.
   - Following it creates an unmarked pair, so a re-run of `lyx beam create` skips step 2 and never writes the mark.
   - Card 19's orch guard and card 23's batten guard then refuse the beam's worktree for good.
   - Fix: step 2 calls `MarkBeam` (card 2) over an existing unmarked pair, or the refusal is worded for this case.

4. **MEDIUM — Card 35 (beam-promote), D10 step 1.** An unreachable origin leaves the promote with no defined next step.
   - Step 1 reuses `cancelRun`'s first half, whose `judgeOrigin` makes the cancel end-only when origin is unreachable. The card does not say what promote does then.
   - `internal/battencli#cancelRun`'s end-only branch stops after ending the run's processes, before the records and the claim.
   - If promote continues, it writes the mark (step 2) with origin unreachable.
   - If it stops, no refusal names a re-run of `lyx beam promote <slug>` once origin answers.

5. **MEDIUM — Card 27 (batten-cleanup-skip), D16.** For a run read only from the records HEAD, the new way forward can hit cancel's "nothing left" refusal.
   - Example: the row "committed run is running".
   - `internal/battencli#refuseNothingLeft` (cancel.go, about lines 370–410) counts the records only when the directory exists on disk.
   - With the run directory deleted and no claim, pair, session or marker, `lyx batten cancel <slug>` refuses "nothing of <slug> is left to cancel".
   - So the skip's `lyx batten cancel <slug>` does not progress for the committed-status branch.

6. **LOW — Card 35 (beam-promote), step 1.** The refusals promote inherits from `cancelRun` give their re-run as `lyx batten cancel <slug>`, not `lyx beam promote <slug>`.
   - These are the probe refusals and the interrupted-marker refusal; `rerun := cancelCommand(slug, remove)` is in `internal/battencli#cancelRun`.
   - Following that text cancels the run instead of finishing the promote.
   - It still progresses: a later promote accepts the task, since it keeps its owner. But the promote is never named.
   - The card also does not say whether the cancel marker taken by the first half stays present between steps 1 and 3 when promote stops there.

Tool calls:
- Read `internal/battencli/cancel.go:366-418`, `refuseNothingLeft`, to check whether cleanup's new way forward survives cancel's nothing-left refusal; my context had only `cancelRun`'s body, not this function.
