---
round: 2
focus:
  - "D6 in decision-record.md: check that the stated release bound for a removed holder holds against `sweepLoop` in `internal/creelengine/sweep.go`, which restarts its timer at the 30 s base on a changed mail sequence (every post, reply, forward, return, inject) and after a failed listing; settle whether the Decision and its Acceptance criteria name the conditions the bound needs and what a pending confirm interval does under those resets."
  - "D6 in decision-record.md: the sweep that first misses a removed holder sees a changed strand-name set; settle whether the Decision says the confirm interval replaces the wait that sweep would otherwise set, the changed-set base reset included."
  - "D4 in decision-record.md: `Serve` in `internal/creelengine/server.go` takes the hub lock before it folds the journal and binds the socket, and a spawned daemon that loses the start race exits with `ErrDaemonRunning` at once; settle whether D4's exit signal and single last connect still succeed when the winner holds the lock but does not yet answer, and whether the Bound states that exposure."
  - "D9 in decision-record.md: `startupGateNeedles` and `gateAcceptNeedles` in `internal/shuttleengine/claudeengine/startup.go` classify the folder-trust gate and the bypass-permissions gate; settle whether D9's Bound states which gates the new seam lets through, who may call it, and what bounds the trust-list side effect per call."
  - "D1 in decision-record.md: `foldJournal` in `internal/creelengine/fold.go` skips the newest file's last non-blank line when it fails to unmarshal into `record`; settle whether D1's cut test for the repair uses that same parse test, so the skipped line is always the cut line."
---

Scope's same-commit doc list in decision-record.md is also worth one look: `internal/creelcli/doc.go` and `internal/shuttleengine/doc.go` describe surfaces that D2, D4, D7 and D9 change.
