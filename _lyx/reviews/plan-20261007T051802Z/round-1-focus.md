---
round: 1
focus:
  - "Test economy across cards 1, 2 and 3: each plans its own `lyx webster validate` drive in `internal/webstercli/cli_test.go` beside a package-level test of the same check. Settle, per card, whether an existing cli test or a sibling card's drive already covers the cobra-wiring behavior, and name that test."
  - "Test economy in cards 5, 6 and 7: each plans a websterengine test, a cobra-command test in `verbs_test.go`, and sometimes a third at `recoverbatch_test.go`. Settle whether any new function repeats behavior an existing test (for example `TestBeginBatch_PreExistingReportRefused`, `TestRebaseline_RefusesChangedBegunCardBody`) already pins, or is a per-symbol test a public-surface test covers."
  - "Fidelity of card 1's edit to `contracts/stencils/loom/loom-rubric-plan-review.md`: settle whether a Decision or Constraint in `_lyx/discussion/decision-record.md` licenses that file, or whether it is scope the answer key does not license."
  - "Fidelity of card 8 against D9: settle whether moving `writeBackgroundShellFrictionNote` to the points where the outcome is known, and the per-outcome 'what ends it' clauses, stay inside what D9 licenses, and whether every D9 element (turn end counted after the bound, shell not stopped, what ends it, run outcome) reaches both the note and the `RunResult.Warnings` entry."
  - "Fidelity of cards 7 and 6 against D4 and D6, including the uncheckable-record variants: settle whether each way-forward variant the record names (plan-edit, `run --fresh`, `recover-batch` first) is carried by a card with a test that reaches it, and whether card 7's `webster-template-master.md` edit is licensed."
  - "Attack-surface question for the guards that card 3 (Delete downgrade at dispatch), card 6 (restamp of a terminal batch's card hash) and card 10 (leased remote update) loosen or add: settle whether each card's `**Intent:**` states what it can now let through and what bounds it, in terms the decision record's Bound paragraphs support."
  - "Granularity of cards 9, 10 and 11, and of card 4: settle whether the new `gitrepo` methods and `Fabric.RemoteOnlyCommits` are public-surface symbols that earn their own card or helpers that belong in the consuming card, and whether card 4 bundles independently reviewable units."
  - "Verify coverage: settle whether every package any card targets, including `internal/fabricengine`, `internal/landingshed`, `internal/loomcli`, `internal/gitrepo` and `contracts/...`, has its tests run by the `## verify:` section, hermetic build-tagged tiers included."
---

Each entry names a place to look and a question to settle.
None pre-states a verdict.
The `Do not flag` list was checked for each: none concerns a check the gate enforces, a missing `DependsOn`/`Produces`, or an `ImpactSummary` on a non-Edit/Delete card.
Card 1 carries no `planparser/doc.go` check-list row, and no entry raises one, because that file lists no per-check rows today.
