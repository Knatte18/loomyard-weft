---
round: 2
focus:
  - "Finalize entry in `internal/landingshed/finalize.go`: shed persists `current_producer` as the next row before calling it, so settle whether the plan's `finalizeEntered` and the D8 ended-at-Finalize Stuck-with-Halt can fire for a run another machine landed, and which test row reaches it over a real status file."
  - "Every plan card that names a way forward for a stop or failed-push class (ended, gone, rewound, diverged, records conflict, reconcile error, ended-busy): settle whether the named command exists, can push or remove what it is named for (`lyx batten cleanup`, `lyx cleanup --apply`, `lyx fabric push`, `lyx board sync` each have a bounded reach), and succeeds rather than repeating the same stop."
  - "Cards that retype or re-sign a symbol (for example `LandDeps.Pull`) or call a new seam: settle whether every literal, caller and test that fills or reads it, including those in `internal/testkit` and `internal/shedrecipe`, has a card, and whether its package is in the `## verify:` section."
  - "Cards that introduce an injectable deadline, sleep or paused-state read: settle against `PATTERN-test-speed` and the real gate or wait ordering (`gitexec` remote deadline, network gate wait before the call) whether the seam is a field or parameter, is read at call time, and has a card targeting the package that must change."
  - "Every fabric push path (`pushClassified` callers, `PushWarpRebaseFreeAt`, `Fabric.PushWeft`, `PushWarpAt`, Publish's task-branch push) and the local-mode Remove path: settle whether each runs the gone and rewound checks and records a failure, or is excluded by an explicit statement, and whether local mode performs no origin step including the deferred `respawnPairPush`."
---

Round 1 was not converged; the focus entries direct the reviewer to the classes of defect that carried the gating findings.
