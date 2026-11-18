---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
---

# Round 2 fixer-report — Discussion decision record

## Changes

- F1 (fixed): D3 gains a bullet saying a reach turn's failing result goes to the writer alone.
  `gatefixer.WrapRowGate` gains a check on the entry's own gate closure, and on its forked closure when set, that spawns no cold fixer for a result whose findings are the reach note.
  The bypass covers only that result.
  A second bullet says the wrap marks its re-run after a fixer and the gate adds no failure to its count for it, so the attempt count is one per shuttle arrival;
  the attempt-budget sentence now counts failing shuttle arrivals.
  The third D3 criterion gains the Darn-Write case with `darn.gate_fixer` set.
- F2 (fixed): the six edits of the added entry "Mutation check order: trial before the mutation step (F6)" are applied, and the entry itself is unchanged.
  - Goal: the new build order, plus a sentence saying this run builds up to `lym gate mutate`, with the trial and D10 after it.
  - Scope In: the `reach` step on its own line, and the `mutation` step and its rubric note marked as designed here and built after the trial.
  - D10: an opening sentence saying it is built only after the trial pays, in a later darn the parent opens from the D11 board note's decision.
  - D11: it prints D9's survivors, uncovered blocks and packages not judged, in D10's form.
    The template sentence is dropped, the board note asks whether to build D10 and list `mutation`, and the rationale says the step is not built before the trial pays.
  - D12: the config check accepts `reach`; the acceptance of `mutation` and D10's refusal come with D10's later build.
  - Constraints: the `PATTERN-refusal-way-forward` bullet is narrowed to D6 and D8, and the tolerant-records and stencil-ownership bullets drop their counts.
  - Open risks: the trial-order bullet is deleted.
  - Criteria: both D10 bullets are marked as checked at D10's later build.
    The D11 criterion names D9's findings, the template `Review:` criterion now names the D11 board note, and the D12 criterion drops `mutation`.
  - The `PATTERN-test-speed` change and the plan-writer note drop the mutation report.
  - Support log: a Round 2 entry records the ruling, and the rejected-alternatives entry on cutting the mutation check and ledger Q10 now name it.

## Self-check

- fix-way-forward, MEDIUM: the gate cannot tell an arrival from the wrap's re-run. Fixed: the wrap marks its re-run, and the gate adds no failure to its count for a re-run.
- fix-proposal, MEDIUM: the same finding as fix-way-forward's, merged with it. Fixed by the same change.
- fix-consistency, LOW: the attempt-budget sentence counted failing results while the new bullet counted arrivals. Fixed: that sentence now counts failing shuttle arrivals.
- fix-claims, MEDIUM: `SkipForked` covers forked failures alone, and Darn-Write sets no forked closure. Fixed: D3 says the wrap gains a check on the entry's own gate closure, and on its forked closure when set.
- fix-proposal, LOW: the Goal's build order read as if this run built the trial and D10. Fixed: a Goal sentence says this run builds up to `lym gate mutate`.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
The parent's earlier ruling on round 1's F6, recorded as an added entry, is carried out under F2 above.
