---
round: 2
focus:
  - "internal/runpusher/parent.go (settleOnOrigin, dropLandedElsewhere, pushParent): for each step that removes or drops a landing, does a failed reset, hand-over or record write get retried and reported, and does any refused-push record outlive the cause it names, including for a run owned on another machine?"
  - "internal/runpusher/parent.go (refreshTips, pushParent): can a SHA the parent's head no longer carries, after a rebase or reset, still be pushed to origin or mark a run done?"
  - "internal/loomcli/landingdeps.go (RecordLanding, recordHeldLanding): across a Finalize resume or repeat, can a records commit reach origin before the parent on origin holds the landed work?"
  - "internal/beamcli/dobbydeps.go (restartHubPushers, parentTargets), internal/tenderengine/resume.go (pairsToPush) and internal/tendercli/deps.go (PushDeferred wiring): after a cleared refusal, a reboot, a pause or a hub lift, what starts the pusher for each kind of target (prime, beam parent, pair with or without a batten run)?"
  - "internal/webstercli/reset.go: is the records push after a reset one whose owed failure is recorded and holds the run, and do pattern/PATTERN-pair-remote.md and pattern/PATTERN-run-fsm.md say what the code does?"
  - "internal/fabricengine/pull.go (primeAheadByOwedTips, pullWarpArm): with owed landing tips on the prime, which of ahead, behind and diverged read as a successful pull?"
  - "internal/fabricengine: after the push paths changed, is any exported function or error sentinel there or in internal/battenshed left with no production caller or producer?"
---

The entries name the areas whose behaviour a fix is likely to change.
