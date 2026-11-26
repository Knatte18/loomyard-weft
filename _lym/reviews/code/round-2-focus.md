---
round: 2
focus:
  - "internal/fabricengine/remoteteardown.go (rerunWayForward, failedRemoteSteps) against internal/battencli/cancel.go (refuseNothingLeft, removePairAndBranch): for each state a pending remove record can be listed in, including only the pair's records branch left on origin and nothing left anywhere, does the command the record names run to completion and finish what is owed?"
  - "internal/fabricengine/remoteteardown.go (recordsArchiveWayForward): is the way forward and its comment true for a removal that carries a caller's rerun when origin holds no branch of the pair and the cancel's records and claim are already cleared?"
  - "internal/fabricengine/remove.go (removePair, the ErrPairNotFound path) and clearPendingTeardown: with the pair gone locally and on origin, is the pending record retired after the named re-run, or does lym fabric status keep listing it? Check that a test drives this state."
  - "internal/fabricengine/doc.go, the paragraph on a removal's rerun command: does it document only fabricengine, as PATTERN-code-conventions requires, and is every statement in it true of the code?"
  - "pattern/PATTERN-pair-remote.md, the Pair teardown background: does its description of the chokepoint guard match the guard in internal/pairteardown?"
  - "Tests for the caller's rerun reaching its destinations: Topology.Remove's held-teardown-lock refusal (internal/fabricengine/remove.go) and the pairteardown remove closure's forwarding of Request.Rerun (internal/pairteardown/teardown.go). Would a test fail if either dropped the rerun?"
  - "_lym/landing/summary.md: do the What-changes bullets and the Merge danger blast radius match the diff, covering every module it changes (fabric, pairteardown, battencli)?"
---

The entries follow the path of a caller-named re-run from the caller to the record, and then what that command does in each state left behind.
