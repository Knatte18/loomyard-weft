---
round: 1
focus:
  - "internal/loomcli/sharedbootstrap.go ensureStatusStrand, as reached from restoreStatusPane (wiring.go): the replace branch logs a ReplaceStrand failure and returns nil. After a reboot the dead strand is likely on that branch. Settle whether a failed replace on resume leaves the envelope without status_pane, and how that stands against the board entry's expectation and the claims in doc.go, the resume help text and the change description."
  - "The statements about who calls ensureStatusStrand and what it touches: the sharedbootstrap.go file header, its doc comment and its log messages that speak of start or this run. Settle whether each is still true now that resume calls it, and sweep the resumeCmd Short, doc.go and docs/overview.md for other statements the new behaviour falsifies."
  - "resumeFromRunState paths that return without a restore, set against the board entry: the running run with a live driver or held lock, and the halted run refused for no live LLM driver. Settle whether a run in the entry's situation can reach one with a dead status strand and be told nothing about the pane or how to restore it."
  - "The restore on both wake paths: reed.Up now runs after driverLive, the park marker and the driver row were read. Settle whether Up on a rebooted or warm session (revival of earlier worktrees, reconcile, pane bindings) or the status strand replacement can change what the later spawnGoDriver or resumeParkedDriver relies on, and whether doing this under the bootstrap lock is safe."
  - "Compare restore with the steps start runs around ensureStatusStrand (reed.Up, FollowRun, spawnWatchdog). Settle whether leaving out the watchdog spawn after a reboot leaves the resumed run's session short of something it needs, given the answer key's expectation of a restored run view."
  - "A failed restore followed by a refusal (spawnGoDriver error, busy or failed parked-driver delivery): the warning reaches only succeedAfterWake. Settle whether the operator learns of the missing pane on those paths."
  - "Coverage of the production seam: the mutation report's survivor is in loomCLI.wire's restoreStatusPane closure, and TestResumeVerb replaces that seam with a fake. Settle whether any test, smoke tests included, asserts that a resume over a dead or absent status strand brings a live one back, and whether one could."
  - "The Loosened-Check trailer and the select-window setup added to TestSmokeStatusStrandAcrossDriverSeeds: settle whether the trailer names a measured cause and whether the subtest still checks what its name says."
---

Seed pass over `git diff 548291383ceb71438c6855d02d06553ba9c56532..HEAD`.
The answer key is a decision record alone, holding the board entry with no added entries.
The entries name where to look; none states a verdict.
