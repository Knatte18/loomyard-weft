# Fixer report — plan, round 1

Every finding's premise held against the code at HEAD; each was fixed as proposed.
`lyx loom validate-plan` passes over the edited plan, after the fixes and again after the self-check edits.

## Changes

- **F1** (BLOCKING) — card 14: `**Create:**` `plan:internal/arbiterstop#HandsBack`, true when `landingshed.HandsBack` is or the reason opens with `shedadapters.DecisionHaltPrefix`.
  `loomCLI.specFor` and `loomCLI.goDriverDepsFor` join `**Edit:**` and pass it as `Spec.HandBack` and `Deps.HandsBack`; `battenCLI.wire`'s seam reads it.
  `TestCheck_RefusesLandingHalts` asserts `HandsBack` true with the prefix and false without it.
  The bound names the Go driver's tackler beside the arbiter and the re-step, and the ImpactSummary follows.
- **F2** (MEDIUM) — card 12, Carry-over: a `decision`-cause accept also carries every BLOCKING review finding, the brief's `blocking_findings`, not already carried by key, keyed by id, with its class, severity BLOCKING and rounds `[round]`.
  Test row 7 gains an admitted accept over a re-judged CONTINUE whose ledger closes the BLOCKING `design` finding, wanting it still carried.
- **F3** (MEDIUM) — card 16: `fixedGateTierLists`, `fixerTestRow`, `newFixerRunFixture`, `TestLandingDeps_VerifyCommandReadsPlanAtCallTime`, `verifyGateConfig` and `gateScenario.restart` join `**Edit:**`.
  The closing bullet now also covers string literals of the old row name and gate key in test fixtures, and keeps fabric's `merge-in` verb literals.
- **F4** (LOW) — card 12, Routing: `standsAsRecorded` is false for a `decision`-cause round only while its decision file is absent or not settled, so a settled accept stands as recorded on a replay.
  New test row 18: a replay over a settled `decision` accept with a BLOCKING `decision` finding returns Done with no re-judge.
- **F5** (LOW) — card 12: the held-refusal re-escalation and `renderDecisionDetail`'s `refusal` field add, over a dropped-key refusal, that a `continue` meets the same check, halts the segment at its budget, and `lyx loom resume` then re-judges the round.
  Test row 8 gains the dropped-key held case.
- **F6** (LOW) — card 13, What it can skip or let through: states the bound on card 12's handled-false paths.
- **F7** (LOW) — card 10, no-parent bullet: a writer the `discussion` gate re-prompts re-reads the board for a `Decision for <slug>:` note before it keeps the stop, and carries on when it finds one.
- **F8** (LOW) — card 5: `pattern/PATTERN-cwd-resolution.md` joins `**Edit:**`, with a bullet dropping `parentreview` from its Bound packages list.
- **F9** (LOW) — card 3: drops the Create of `TestDiscussionWriteRow_GateListIsDiscussionAlone` and its sentence; `TestSequence_FullRunBlocksAtMergeIn`'s existing table-gate assertion now wants `[]string{"discussion"}`.

## Self-check

- fix-affected-set, MEDIUM: card 16's widened bullet reached `Merge-In` and `merge-in` test literals outside its `**Edit:**` list — fixed as proposed: `TestClassifyRecords`, `TestInnerRun_HaltedStatesWaitExemptWithoutSpawning`, `TestRowEntry`, `TestSettledSegmentGuard`, `TestFinalizeDirtGuard`, `TestParentLockWaiting` and `TestWriteRunRecord` join `**Edit:**` and the bullet names them.
- fix-consistency, MEDIUM: card 13's bound claimed a brief on every handled-false path, but an automatic `budget` decision file writes none — fixed: the bound now separates the escalation-record and parent-decision paths, whose brief points at the review, from the `budget` path, where the findings reach the parent only through the review in the run records.
- fix-consistency LOW and fix-proposal MEDIUM, merged: card 10's carry-on step said "as for a parent answer", which brings in the ask-your-parent `lyx loom decision add` step that a writer with no parent cannot take — fixed: the writer cites the note in the record and names the answer as an unrecorded parent correction in the change description.
- fix-consistency LOW and fix-proposal MEDIUM, merged: card 10's "or a resume reaches you" contradicted the card's bound "nothing respawns it", and added nothing because a resumed row's fresh writer already reads the notes in Step 3's first bullet — fixed: the phrase is dropped, so F7's fix covers the gate re-prompt only.
- fix-proposal, MEDIUM: card 12's carry-over dedup by key rarely matches, because ledger keys are judge-chosen (`bouncer-template-judge.md:101`) and `ledgerEntry` records no review id (`internal/shedadapters/bouncerfiles.go:175`), so a finding can be carried twice — fixed with my own change: no mapping from ledger key to review id exists to dedup on, so the card now states the bound, that matching is by exact key and a finding keyed apart is listed under both identities, repeating a risk but dropping none.
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.
