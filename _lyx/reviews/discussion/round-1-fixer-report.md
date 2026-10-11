# Fixer report — discussion round 1

Every finding was checked against the code before it was fixed.
`lyx loom validate-discussion` passes on the edited pair.

## Changes

- **F1** (D5's CIRCLING premise): confirmed in `internal/shedadapters/bouncerprompt.go` (`decisionRuleMarker`), where only a `design`-class finding gates and earns CIRCLING.
  D5 now names the path by class and severity: a design-class finding at MEDIUM or worse runs CONTINUE and then CIRCLING from the checkpoint, a BLOCKING finding of another class runs CONTINUE until the bounce budget, and any other finding converges from round 2 on.
  D4's rationale, the D4 Auto-mode assumption and the support log's interview step 6 and its rejected alternative no longer cite CIRCLING as the only path.
- **F2** (closing-round bound, LOW and NIT carry-over): confirmed in `internal/shedadapters/bouncercarryover.go`, where `carriesOver` keeps only BLOCKING and MEDIUM.
  D2's and D3's Bounds each state that in the converging round the fixer's cited evidence is the only bound.
  D5, the Open risks entry and the support log's interview step 4 now say only MEDIUM and BLOCKING closing-round disputes reach `## Open risks`, and that a LOW or NIT one is recorded only in the fixer-report.
  The Open risks entry also says that an every-fix-worse dispute leaves a defect the fixer agrees is real.
- **F3** (two grounds for a Do-not-flag finding): removed "(its `Do not flag` list included)" from D2's evidence list.
  D2 now states that a Do-not-flag finding is disputed as `premise false`, and the Auto-mode assumption points at D2.
- **F4** (write surface in "every change"): D2 states that "every change" includes off-surface changes, and that a finding whose only sound change leaves the surface is deferred under D3, never disputed.
- **F5** (D4 timing in the clean-room order): D4 states that the reviewer amends the matching `**Issue:**` in its saved review before it ends its turn.
  This works because, per `internal/burlerengine/doc.go`, the ready marker is written only after the reviewer is done.
- The first Acceptance criterion now lists F3's and F4's two D2 rules, so the plan writer pins them in `burler-step-3-fix.md`.

## Disputed

None.

## Deferred

None.
