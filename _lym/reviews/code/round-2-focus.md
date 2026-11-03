---
round: 2
focus:
  - "Parent branch: in `internal/loomcli/prepush.go`, `guard` and `ownBranches` keep only branches whose `Slug` is the run's, and `reportOfRun` in `internal/loomcli/landingdeps.go` narrows the run-scope report to the run. Settle whether a move of the run's parent branch on origin reaches a notice or a stop reason, as the board entry asks of the base and parent-branch compare."
  - "Stop reach: `loomCommitStatusDeps` in `internal/loomcli/wiring.go` wires `Spawn`, `Push` and the commit-records `push_stopped` envelope to the pre-push check. Settle whether tests pin that a stopped check makes `Spawn` return nil without a push, `Push` return an error satisfying `errors.Is(err, errPushStopped)`, and `commitRecordsVerb` answer with the `push_stopped` key."
  - "Skip condition: the `skip` closure built in `newPrePushCheck` in `internal/loomcli/prepush.go` keeps the check off pushes-off runs, the prime, pause-covered runs and Finalize. Settle whether a test drives the closure itself rather than a stub."
  - "Failure paths in `stopReason` in `internal/loomcli/prepush.go`: settle whether a failed `landed()` read, a failed notice post and an own `needs_replay` branch each behave consistently with the other failed reads and with `BranchState.stops()`."
  - "Ordering: `loomCommitStatusDeps` in `internal/loomcli/wiring.go` runs the check in the push seams after `fabricengine.CommitAnchoredPaths`. Settle whether the board entry's 'before a run commits for a push' is met, or whether the diff records a decision for the local commit."
  - "Coverage of the PR-Review path: `pullRequestDeps.PushTask` in `internal/loomcli/landingdeps.go` guards with the check, and `PausePair` is new in `internal/tendercli`. Settle whether tests reach both."
  - "Docs: `pattern/PATTERN-owner-machine.md` names the push in its new bullets. Settle whether its verdict sentence, its `## Origin reconcile first` section and its Enforcement line state what the code and tests do."
---

Round 1 judged CONTINUE; this focus names where to look, with no pre-stated verdict.
