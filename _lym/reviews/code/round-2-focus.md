---
round: 2
focus:
  - "internal/creelengine/sweep.go `sweepOnce`, `sweepSubjects` and `releaseGoneHolder`, with `isHubOrchName` and `checkReceiverLocked` in internal/creelengine/pool.go: settle whether mail stored for a hub orch with no listed strand stays unread across repeated sweeps and reaches the strand once it is listed, and whether a holder that is no hub orch is still released as before."
  - "internal/dobbyengine/landnotice.go `takeDoneEntries` with `TaskView.OwnedElsewhere` in internal/dobbyengine/starts.go and `DobbyTasks` in internal/battencli/dobbyreads.go: settle whether only the machine that owns a done flagged entry sends its landing line and clears `notify_on_land`, and whether `DobbyTasks` still returns the other tasks when one entry's parent or owner cannot be read."
  - "internal/dobbyengine/landnotice.go `settleLandNotices` and `landNotices`, with `Deps.ClearNotifyOnLand` in internal/dobbyengine/loop.go: settle whether a failed post, a failed clear after a successful post, a failed beam.yaml read and a nil clear seam each end in at most one line per run and no repeat every tick."
  - "internal/boardcli/cli.go `landedNotice` and the `prune` help, with the prune descriptions in docs/overview.md, contracts/orchrole/orch-role.md and plugins/ly/skills/board/SKILL.md: settle whether the notice appears only for an upsert that sets `notify_on_land`, and whether every text that describes prune says a done entry still carrying `notify_on_land` is kept."
---

Round 2 reviews the fixes to the round 1 findings.
