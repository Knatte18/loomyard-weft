# Round 2 fixer-report — discussion

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  The premise holds: `pollEventsTick` in `internal/shuttleengine/wait.go` holds a non-waiting turn end whose output files are missing, `Wait` logs "turn end held, output files missing" and calls `notifyHeld`, and Master's spec sets no `HoldDeferral`.
  `background.go` counts only `running` payload tasks, and the board entry's pane capture shows the fork reported `finished` at each interim stop.
  D5 now states that premise in its own `Premise:` paragraph.
  Its bound is restated against shuttle's real behaviour:
  an interim notice in the batch loop is a held turn end with one parent hold notice per stop, accepted for this fallback;
  a misread final notice, or a fork that never sends one, is held until `master_timeout_min` or loom's step idle kill.
  Correcting the Master template's sentence on a turn end with no background work and no outcome file is added to D5 and to Scope.
  An acceptance criterion covers the corrected sentence.
  The first Open risk now names the hold notices that D5's fallback produces when the ceiling variable is ignored.
  The support log's question ledger gains Q10, recording the settled premise.
- **F2 (BLOCKING), fixed as proposed.**
  The premise holds: with outcome and summary on disk, a plain Stop is `OutcomeDone`, and `Wait` sends a gated Done to `handleGatedBoundary`.
  D5 drops the claim that skipping the rewrite keeps Merriam off the gate.
  It names the residual case, a fixer interim notice that only a call backgrounded at the one-hour ceiling produces, and states its outcome:
  each arrival spends one `verify_gate_attempts` on a dirty or half-fixed tree, and the run ends stuck within the attempt budget unless the fixer's final notice and a passing fix come first.
  A one-fixer rule is added as a requirement: a gate message that arrives while an earlier fixer has sent no final reply spawns no second fixer, so two fixers never edit the tree at once.
  An acceptance criterion covers the rule.
- **F3 (MEDIUM), fixed as proposed.**
  The premise holds: a transcript that ends in a `tool_use` with no result looks the same whether the call is held or its writer is gone.
  D9 now reads a pending tool call as active only until D1's ceiling has passed since the fork transcript's last write, and after that as that write time.
  Its bound covers a call whose writer is gone: at most one hour of false activity past the last write.
  An acceptance criterion and the Q9 ledger entry cover the cap.

`lyx loom validate-discussion` exits 0 on the edited pair.

## Disputed

None.

## Deferred

None.
