```yaml
lens: way-forward
outcome: reported
```

Task: review the plan through the way-forward lens. For each refusal, stop or recovery step a card adds, check the next step it names and whether that step can succeed from the state it leaves.

**Findings**

1. **MEDIUM**: Card 2, `--take-hold` over another set. The way forward cannot succeed.
   - Card 2 step 5 makes this refusal refuse "as in 3", and step 3's way forward is `lym adopt <hold's subjects>`.
   - In this case the hold belongs to another machine. A plain `lym adopt <hold's subjects>` therefore hits step 4 and is refused naming that machine. The operator loops between the two refusals.
   - The way forward should be the take-hold re-run of the hold's subjects. Only the operator reaches this refusal, because card 4 denies agents the flag.

2. **MEDIUM**: Cards 3, 8 and 9 name a one-subject re-run under a multi-subject hold. Each of these ways forward names `lym adopt <subject>` or `lym adopt <project>` for a single subject:
   - card 3's exhausted takeover retry;
   - card 8's `fixReadError` and `fixFilingError`;
   - `runCheck`'s reconcile-failure and gate-stopped lines (`internal/adoptcli#runCheck`);
   - `resolveSubjectWorktree`;
   - `checkView`'s "re-run lym adopt %s";
   - `addError` (card 9, step 2).

   When this machine's hold covers more than one subject, card 2 step 3 refuses that re-run as "any other set, overlapping included". Recovery then takes two hops through a second refusal. Each of these should name the whole re-run over the hold's subjects.

3. **MEDIUM**: Card 2, `FixHold` when the hold stands for another machine. The way forward cannot succeed.
   - `FixHold` refuses with way forward `lym adopt <subjects>`.
   - If another machine took the hold over, a plain adopt here hits card 2 step 4 and is refused. The draft fix task is also left behind, because `fronttask.File`'s `between` failed.
   - The refusal should name that machine and say this machine's check ends here: wait, or ask the operator.

4. **MEDIUM**: Card 9, step 2 and steps 7–8 conflict on when the hold is re-read.
   - Step 2 says `holdStands` "runs before each step from here on".
   - Steps 7 and 8 then call `runBeamSteps` after `HoldCheck`, and a green `HoldCheck` has already called `ClearHold`.
   - A re-read before the bring-up finds the hold gone and "stops … starts nothing". A green adopt would then leave every subject down, against criterion "the runs go on with no hand lift".
   - The card should say the last re-read comes before the check (step 7), not after the hold's own clear.

5. **MEDIUM**: Card 8, the check that did not run (`Ran` false). Neither end names a next step.
   - For the foreground adopt, card 9 does not say whether it exits ok or non-zero, or what it names.
   - For `checkCmd`, the exit code is unstated. This matters because `fronttask.Settle` keeps a failed action's record and retries it at every settle (`internal/fronttask/fronttask.go:317-319`).
   - A natural way forward: the running check posts its own end, so wait for it, or re-run `lym adopt <hold's subjects>` once it ends.

6. **MEDIUM**: Card 8, failures after a green check have no stated way forward. A failed `ClearHold` (for example origin unreachable), `DropBoltHold` or `BeamUp` inside `runHoldCheck` step 3 is unspecified. The fix tasks are already removed by then.
   - A re-run of `lym adopt <hold's subjects>` would progress: the hold is kept, the check goes green and the hold clears. No line names that re-run, and no row reaches these ends.
   - The `BeamUp` failure has a ready wording in `runBeamSteps`, which names `lym beam start <slug>`, but card 8 does not reuse it.

7. **LOW**: Card 8 step 4 and card 9 step 7/8, the line at the try limit names no next step.
   - The line is said to name "the failing tests and the fix task", nothing more.
   - Today's `failedEnd` limit line ends with "re-run lym adopt %s once you have settled it, whose check removes that task" (`internal/adoptcli#failedEnd`). The card neither keeps it nor states a replacement.
   - Under a kept hold the try counts stay at the limit, so the only way out is a hand fix followed by that re-run. The line should say so.

8. **LOW**: Card 8 step 1, the no-hold refusal reached from the fix task's landing.
   - The refusal exits non-zero.
   - `fronttask.Settle` then keeps the record and re-runs `lym adopt check` at every settle, refusing each time (`fronttask.go:317-319`), until the operator runs the named adopt.
   - Consider exiting ok with the refusal as a line, or saying the record is dropped.

9. **LOW**: Card 9 step 2, the `holdStands` stop names no next step. It names the subjects' owner and starts nothing, but says nothing about what this machine's operator does. It could say the adopt moved to machine X and that nothing more runs here.

10. **LOW**: Card 9 step 4, the `CatchUp` error end is unspecified.
    - The card does not say whether a failed catch-up stops the adopt, or what it names; for an unreachable origin that would be a re-run once origin answers.
    - Card 3 has the same gap for an error from step 1 (fetch) and step 6 (fast-forward) of `TakeOverRun`.

11. **LOW**: Card 9 makes `deployStale`'s stop text false. Card 9 lists `deployStale` under Uses only, but its stop text says "the owner stands and the adopted runs stay paused, so none goes on under the old lym" (`internal/adoptcli#deployStale`). The adopt pause is retired, and the runs are now held by the hold. The text should say the hold stands.

12. **LOW**: Card 9, the `--take-hold` with `--all` refusal points at subjects the operator cannot see from there. Its way forward is "name the hold's subjects", but `validateRequest` runs before any hold read. It should point at `lym beam status`, which shows the hold's subjects after card 6.

**Other, outside my lens**

- Card 9's `movedRun` likely contradicts the row `AReRunUnderItsOwnAdoptFixHoldKeepsKindTaskTriesAndFilesNoSecondTask` ("no `TakeOver` of the fix task's run"). After an adopt from another machine, the hold's `PriorOwner` is the other machine. So on a re-run the fix task's run (under that subject) still reads as moved and gets `TakeOverRun` (archive, reset, takeover entry). That touches it, against criterion "leaves the fix task's branches and processes untouched".

**Calls made**

1. A grep of `internal/fabricengine` for `Refused` handling, and of `destroy.go` for refusal wording. My context lacked whether a refused replay leaves the local Bolt clean (it sets the commit aside under a tag, `replay.go:891-897`) and whether `DropLocal`'s refusals name the retired flags (no such text near `destroy.go:1977`).
