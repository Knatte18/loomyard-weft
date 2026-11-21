---
round: 1
focus:
  - "internal/loomshed/redfirstgate.go: does the hold path (pending answer, HEAD-tree comparison via verifytree.HeadTree, the parent's 'No test seam' record lifting it) spend no verify attempt on the unjudged and no-test refusals, and does the Final closure turn the same refusal into a failure with findings; check every state transition of the hold for a path that holds forever or runs the check twice on one tree."
  - "internal/shedrecipe/entries_gate.go and contracts/recipes/darn-recipe.yaml: the red-first entry is now MayHold with a Final closure; does GateResult.Pending in internal/shuttleengine/gate.go accept it, and do the other plain-gate entries keep their previous contract."
  - "internal/loomcli/darnwiring.go preflightTableError: is the config-or-model-spec clause withheld only from a *stencilstore.NewerBoardCopyError and kept on every other build error, including a wrapped one."
  - "internal/shedadapters/gateattempts.go and the burlerfix, burlerreview, multillm producers: issue #174 asks to reproduce what removes the findings file, the decision record names the orphan scratch sweep in internal/battencli/cleanup.go as a suspect; does the diff address the cause it found, or only drop the deletion on a passing gate, and is the sweep still able to remove the file the halt names."
  - "internal/shedadapters/gateattempts_test.go: a 63-line deletion; check the removed tests covered no behavior the diff still has, and that the retained assertions still check what their names state (a loosened-check question)."
  - "Commit trailers: each of the three fix commits carries a Root-Cause:; judge each against the defect the diff actually removes, and look for any widened or removed assertion in a test that exists at the review base without a Loosened-Check: trailer."
  - "Reach and mutation reports: the new loomshed helpers (holdsForCommitOrRecord, redFirstGate.arrival, exempt, final, verify, redFirstHoldLine) and verifytree.HeadTree show no mapped test; check whether TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands and TestRedFirstGate_VerdictPerLabelRecordAndExitCode cover each behavior, and that the Darn-Write stencil change in contracts/stencils/darn/darn-template-write.md keeps every rule the answer key did not ask to change."
---

Scope is the Scope decision in the record: issues #173 (red-first hold), #175 (newer-stencil way forward) and #174 (findings path that a halt names).
The Goal body is corrupted, so the Added-after-Discussion entry is the measuring stick.
