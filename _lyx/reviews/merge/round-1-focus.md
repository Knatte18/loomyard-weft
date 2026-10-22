---
round: 1
focus:
  - "internal/loomcli/drivewiring.go and internal/sheddriver/drive.go: the parent added `Deps.StartCheck` (wired as `c.finalizeGuard()`) and made `run` read status once before `resumeOpenAttempt`; the task added `registerRunProcess(..., runproc.RoleDriver)` at driver start. Settle whether the registration still happens before the start check can hand the run back, so a refused start does not leave a driver unregistered or a registration unreleased."
  - "internal/boardcli/cli.go: the parent added handle selectors and changed the `payloadKeys.exclusive` type, the task added the deferrable `sync` flag with `pauserecord` gate deferral. Settle whether both sides' edits compile into the behavior each meant, including `sync` payload and help text, and whether `boardengine.DeferrableSyncFlag` callers still match."
  - "The task's run-process registration and ownership calls against the parent's new config-layer and yamlengine changes (profile key set, profile maps merged by name): settle whether any runproc or runprofile code the task touched reads a profile key through a path the parent changed."
  - "The doc.go files of boardcli, boardengine, fabricengine, loomcli, sheddriver and docs/overview.md, both sides edited: settle whether the merged prose still states one true contract, for example the board handle and `StartCheck` text beside the task's runproc and `RestepRecorded` text."
  - "Test economy across the overlapping packages: settle whether the parent's `StartCheck` tests in sheddriver and its handle tests in boardcli leave a gap where the task's new driver registration or deferred sync interacts with them, and whether any merged test now duplicates another."
---

Seeded from the overlapping files and the two sides' diffs of them.
