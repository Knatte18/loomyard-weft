# Round 3 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed: `_lym/plan/12-drift-expect-producer.md`, `TestDriftExpect`.
  Added a row after the missing-baseline-directory row: a scope record with no `awaited_at` is a Stuck saying the capture never counted and naming `lym drift defer`, with no agent spawned and no seal commit written.
  The prefix sentence now covers all three Stucks.
- **F2** (BLOCKING), fixed as proposed, then refined by the self-check.
  - `_lym/plan/16-drift-verify-lines.md`, `TestDriftVerify`: added the interim row. A recorded `command` diff is a Stuck naming `lym drift defer`, its reason opening with `DecisionHaltPrefix` then `DeferHaltPrefix`, with no judge and no compare. The same record, once deferred, is Done.
  - Card 16's Intent now places the interim check right after step 2, so a scope-`off`, deferred or waived record is still Done.
  - `_lym/plan/17-drift-verify-command.md`, `TestDriftVerify` paragraph: it now says card 17 replaces card 16's interim row with its command-mode rows.
- **F3** (MEDIUM), fixed as proposed: moved the `darnWriteRow` and `websterRow` Create lines from card 19 into a new **Create:** group on card 16, placed before **Delete:** and **Extends:**.
  - Card 19 now lists both constants under **Uses:**, after `expectRow`. Its Intent reads "this card's `recheckRow` and card 16's `darnWriteRow` and `websterRow`".
  - Card 16's Intent says the route is derived by comparing `Deps.Rows` against those two constants.
  - Card 16's ImpactSummary names the two constants (self-check).
- **F4** (LOW), fixed as proposed, then corrected by the self-check: `_lym/plan/15-drift-judge-seat.md` gains a Bound after the exempt-paths paragraph. The Bound says:
  - an edit under the exempt run directory, friction directory or archive siblings goes unseen by the guard;
  - those paths hold no code, expectation, `scope.json` or `judged.json`;
  - loom reads the run directory's `status.json` and settle records, and the stop report reads the friction notes;
  - containment reads only the status row from the run directory, which loom rewrites and commits at the row's transition, so no judge edit there decides containment.
- **F5** (LOW), fixed as proposed: card 5's `TestParseExpectation` gains rows for an `n` below 1 and a case-level `why` under `lines`. Card 6's `TestParseJudgement` gains rows for a ruling outside the mode's set, an empty reason, a repeated case under `command`, a `cells` list under `command`, a `cases` list under `lines` and an unknown field. All are rows of the existing tables.

## Self-check

- fix-consistency, LOW: card 16's ImpactSummary did not name the new row constants. Fixed as proposed.
- fix-claims, MEDIUM: card 15's Bound claimed containment reads nothing under the exempt paths, but it reads the status row through `Deps.Status`. Fixed as proposed: the Bound now names the status row and why a judge edit to it decides nothing.
- fix-proposal, MEDIUM: the F2 interim Stuck as first written stopped a deferred run again. Fixed as proposed: the check sits after step 2, and the row asserts that a deferred record is Done.
- fix-proposal, LOW: card 15's Bound left out the run directory's settle records. Fixed as proposed.
- fix-proposal, related note: `Defer` first admits a Drift-Verify halt only in card 19, so `lym drift defer` is refused there during cards 16 to 18. Deferred; see below.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- The fix-proposal fork's related note: through cards 16 to 18, every Drift-Verify Stuck that names `lym drift defer` (the interim `command` Stuck, and step 3's never-counted and baseline-fault Stucks) names a verb whose Drift-Verify admission card 19 adds later.
  This predates this round's change set; the step-3 Stucks were not touched by any finding.
  Settling it needs a schedule decision across cards 16 to 19: move `Defer`'s wider admission ahead of card 16, or accept the interim gap with a stated bound.
  That is a plan-level choice beyond this round's findings, so it is left for the next review round to raise.
