---
round: 1
focus:
  - "internal/loomcli/stencilpins.go `warnChangedStencils` reads pins with `c.runID`, while `writeStencilPins` writes under `shedrun.SelfRunID`: settle whether both resolve to the same record directory for a default run and for a run with a non-default run id, via `RunDir` and `runSegment`."
  - "internal/loomcli/sharedbootstrap.go step 2d: settle whether pins are written on the first seed only, as the Goal says (a re-entry keeps the record). Check what `wroteStencilPins` does when the record already exists, and whether a failed pin write can leave the seed commit's `commitPaths` naming a file that was never written."
  - "internal/loomcli/start.go and resume.go: settle whether the Goal's 'resume compares and warns the orch' holds on every resume path that spawns or wakes a driver, and whether `start` comparing is licensed by the Goal. Check the no-op success paths (done run, live driver) for scope."
  - "`warnChangedStencils` posts a parent notice on every call: settle whether repeated starts or resumes re-send the same notice, and whether that follows the Goal's 'warns the orch of each that changed'."
  - "internal/stencilstore/pins.go `Pins` and `ChangedPins`: settle whether pinning every registry stencil matches 'each stencil it may render', whether a missing board copy at seed time and a later install is handled as the doc comments claim, and whether `BodyHash` is the digest `lym stencil diff` compares."
  - "internal/shedrun/stencilpins.go: settle whether the record is tracked and committed as its doc says, the lock path is right (`lock.TrackedFileLockPath`), and the new run-directory file collides with no existing run-directory enumeration or cleanup."
  - "Check the `notifyParent` wiring in `newLoomCLI` (`orchcli.NotifyParent` with `loomProgram`) for a new import cycle, a nil `notifyParent` in test-built `loomCLI` values, and the failure path when the run has no parent."
  - "Tests: check each new test against the Test economy bullet. In particular, settle whether `resume_test.go`'s two-line change and `TestResumeVerb_NamesStencilsChangedSinceTheRunStarted` cover the start-verb path the diff adds, and whether any new behaviour has no test."
  - "Check that the doc and help text the diff changes (`loomcli/doc.go`, `shedrun/doc.go`, `stencilstore/doc.go`, the start and resume long help) match what the code does, and that the repository's constraints document asks for no other doc to land with this kind of change."
---

The Goal is the only measuring stick: the decision record's other sections are empty, so there is no plan or settled Decisions list.
