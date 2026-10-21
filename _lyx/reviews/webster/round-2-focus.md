---
round: 2
focus:
  - "internal/fabricengine/replay.go, replayWeft and replayBolt: when set-aside tagging fails, settle whether the branch and the set-aside commits end in a state recoverable without the reflog."
  - "internal/fabricengine/pushclassified.go and coalesce.go (pushSideDetached): settle whether every refused push, including one matching no failure class and one from the detached child, ends in a failed-push record and one creel notice to the orch, as the declared PATTERN-push-failure-visible text requires, and whether TestPushFailureClasses pins each path."
  - "internal/landingshed/stuck.go (reconcileVerdict) and the runReconciler closure in internal/loomcli/landingdeps.go: settle whether the verdict judges only the run's own entry of the run-scope report, and whether a test covers a stopped parent entry beside an active own entry."
  - "PATTERN.md and pattern/: settle whether every entry outside the two declared ones reads as on main, and whether the two declared entries carry their declared text."
  - "internal/fabricengine/pull.go (Pull) and destroy.go (cherryPickOnto): settle whether a weft infrastructure error and a ResetKeep error that is not a local-change refusal are reported as what they are."
---

Entries name where to look and which question to settle; they cap no severity.
