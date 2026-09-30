# Fixer report — discussion round 1

All seven findings fixed.
`lyx loom validate-discussion` exits 0 on the edited pair.

## Changes

- **F1 (MEDIUM), cascade risk.**
  Added a Decision: the llm-arm removal is non-recursive, so reed refuses a status strand with children and the warn-and-continue path handles it.
  Rewrote the Open risks bullet to that residual risk, with the evidence that no current session holds such a strand.
  The removal acceptance criterion now says "non-recursively".
  Support log: new rejected alternative (recursive removal) and ledger Q10.
- **F2 (LOW), sidecar delete.**
  Removed the sidecar Decision and the sidecar clause from the removal acceptance criterion.
  Support log: interview turn 7 and ledger Q7 record the reversal; the "leave the sidecar" rejected alternative is replaced by "delete the sidecar", rejected under YAGNI.
- **F3 (LOW), invariant sub-bullet.**
  The amendment Decision now states that the first sub-bullet stays unchanged and why the removal is strand-set selection, not a behaviour gate.
- **F4 (LOW), Goal overstatement.**
  The Goal is now qualified by "Once `lyx loom start` has run on this build".
- **F5 (LOW), untestable criterion.**
  Deleted "`step` does not read the seed's driver"; the step criterion now requires identical behaviour on `go`- and `llm`-seeded runs.
- **F6 (LOW), failed reed Up.**
  Added a criterion: a failed `Up` fails `start` on either arm before the watchdog and driver spawn, and fails `step` with the bootstrap refusal kind.
- **F7 (NIT), helper-boundary Decision.**
  Cut the "`ensureStatusStrand` keeps its name and body" bullet; its requirement lives in the `start` split and `step` Decisions, and its deliberation in interview turn 5.

## Deferred

None.
