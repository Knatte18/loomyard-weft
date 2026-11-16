# Round 2 fixer-report — discussion

Every non-`decision` finding is fixed in `_lym/discussion/decision-record.md`, and F17 also in `_lym/discussion/support-log.md`.
F10 and F11 are `decision`-class, held by the parent, and left out here.

## Changes

- F1 (BLOCKING): D12 now attaches through `AttachGated` with the strand's output file and gates for every attempt in flight, before it records anything.
  It records a report only from the Wait's result.
  The D12 criterion adds the case of a crash while a strand is re-prompted on a failing gate.
  Fixed differently in one part: the proposal records a report directly from disk when the attach answers not found.
  I did not make that change, because a run that reached `OutcomeDone` has its directory removed by finalize (`internal/shuttleengine/attach.go:525`).
  An attach after a crash at that point answers not found and would record a report with failing gates.
  Instead, D12 counts that attempt a machinery loss and never records it done.
- F2: D3 numbers each batch by its first card, as the shared batch identity does, and gives pending batches no batch record until they begin.
  D11 hands the shared checks the recorded partition mapped onto the current plan.
- F3: D6 stops the strand of every attempt that ends without being recorded done, whatever ended it.
  D13 drops the claim that holding the notice back keeps a strand from acting and points at D6's stop instead.
  The D6 criterion adds the machinery-loss and Weaver-fault cases.
- F4: D12 gives every verify fixer strand an identity, its own output file, recorded in flight before its spawn.
  D10 names that file as `fix-<n>.md` in Weaver's reports directory.
  The D12 criterion adds the crash-during-fixer case.
- F5: D2's reset moves the code branch back, restores the plan directory in the weft from the weft commit the first begin records, now added to D3, and commits it through fabric.
  It also archives `_lym/weaver/` before clearing it.
  The false plan-returns claim is dropped, and D9 now says weft branch.
  The reset criterion checks that the pending cards read as at the run's start.
- F6: D2's reset also refuses while an attempt in flight has a live strand, and stops no strand on its own.
  The reset criterion covers it.
  The self-check added the `--stop-strand` way forward.
- F7: D7 bounds the no-spawn path against the row's `gates` entries.
  Fixed differently: the proposal let a batch take the no-spawn path unless its last attempt had failed on a gates entry.
  The self-check's case breaks that: the arbiter's fix can break a gates entry that nothing then runs.
  So a row with `gates` entries never takes the no-spawn path, and its stopped batch gets the last-rung attempt, whose strand runs those entries.
  The D7 criterion covers this.
- F8: D7 has a `NEEDS_PARENT` re-entry start the next attempt on the rung that asked, never on the no-spawn path.
  D5's bound and the auto-mode assumption match, and the D7 criterion covers it.
- F9: D7's re-entry re-reads the plan, restamps Weaver's baseline and maps the recorded partition onto the current plan before any check.
  D11 says shared refusals name Weaver's way forward under Weaver.
  The D7 criterion adds the arbiter's pending-card edit.
  Fixed differently in one part: the proposal refuses whenever the stopped batch's own cards changed, naming `lym weaver reset`.
  That refuses the natural answer to a `NEEDS_PARENT` question, a card edit, and the reset reverts that edit (D2).
  Re-entry now carries a card-text edit into the next attempt.
  It refuses only when the plan's card set no longer matches the recorded partition, the case `mapPartition` refuses (`internal/websterengine/partition.go:131`).
  It names putting the card set back, or `lym weaver reset`.
- F12: D9 adds `outcome.yaml`, written done when the row returns Done.
  D13's merge-in bullet counts a Weaver run in flight as `RunInFlight` does for Webster.
  The criterion covers a finished run.
- F13: D6 records the outcome first and then stops the strand.
  D12's re-entry first stops the live strand of every attempt whose outcome is recorded.
  This is generalized from "recorded failed" to any recorded outcome, because F3 now records every non-done end.
- F14: Fixed differently. The proposal bounds move-aside by "HEAD is the run's start commit".
  The start commit is in the corrupt state file itself (`bs.StartSHA`, `internal/websterengine/auditledger.go:248`), so nothing can check that bound.
  D1 now drops move-aside: a corrupt state file never counts as missing, and the way forward is repair, which for Weaver's state file can restore it from the per-batch weft commits.
- F15: D12 sends only the decisions that state does not record as delivered.
  D13's live notice records each delivery under Weaver's state-mutation lease, and D9 lists the field.
- F16: D5 lists `verify_gate`.
- F17: D11's signature line moved to the support log's Turn 2 as evidence for D11.
  The review said Q12, but Q12 is the D13 seam; the self-check caught it.
  The requirement stays in D11.
- F18: D11 says every told `Geometry` directory naming Webster's run, reports, prompts or scratch directory names the active implementer's, so Weaver's locks sit under `.lym/weaver/`.

## Self-check

- fix-way-forward MEDIUM, D7's own-cards refusal named a reset that reverts the edit: fixed with my own change, merged with fix-proposal 2 (see F9 above).
- fix-way-forward LOW, D1's restore-from-weft way forward does not exist for Webster's state mid-run: fixed; D1 names restoring only for Weaver's per-batch commits, and repair otherwise.
- fix-claims MEDIUM, the same D1 claim: merged with the item above, fixed.
- fix-claims MEDIUM, D6 does not say what happens when the state write fails: fixed as proposed, merged with fix-proposal 3. The strand is stopped all the same, the row returns Stuck naming the fault, and re-entry counts it a machinery loss. The D6 criterion covers it.
- fix-claims LOW, D12 overstated the gated Wait: fixed; it now waits until the gates pass or the re-prompts run out, the latter a D6 failure (`wait.go:171`).
- fix-consistency MEDIUM, D11 promised Weaver verbs that D2 does not add: fixed; D11 names D7's way forward for a plan-partition refusal.
- fix-consistency MEDIUM, D7 mapped only the done and stopped batches, against D11: fixed; D7 maps the whole recorded partition.
- fix-consistency LOW, the support log's Turn 4 D12 bullet was stale: fixed with a round-2 revision note.
- fix-consistency LOW, Q6 was stale against the new D7: fixed.
- fix-consistency LOW, the "(evidence for Q12)" cross-reference was wrong: fixed to D11.
- fix-proposal MEDIUM, D12 recorded a report from disk without a gate verdict: fixed with my own change (see F1 above).
- fix-proposal MEDIUM, D7's own-cards refusal discards the run on a card edit: fixed with my own change (see F9 above).
- fix-proposal MEDIUM, D6 state fault blocks the stop: merged with the fix-claims item above, fixed.
- fix-proposal MEDIUM, D7's gates-entry bound claims a pass nothing checks: fixed with my own change (see F7 above).
- fix-proposal LOW, the reset's in-flight refusal offered no way to end only the strand: fixed; `lym weaver reset --stop-strand` stops it through shuttle's stop verb, then resets.
- fix-proposal LOW, the reset silently reverts approved Plan-Review edits: fixed; D2 now states that the reset reverts every plan edit since the run's start.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
