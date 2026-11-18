---
round: 2
focus:
  - "internal/shuttleengine/relaunch.go, relaunchForReprompt: the resumed strand reuses the run's events file; settle whether awaitStartup in wait.go can mark the new strand started from the earlier session's session-start and turn-start lines before the new pane is ready."
  - "internal/shuttleengine/relaunch.go, relaunchForReprompt and each engine's Prepare: settle whether the resumed session keeps the settings, session scope and standing passages the original launch had, given that the resume alters the spec's Skills."
  - "internal/shuttleengine/relaunch.go and run.go: settle whether the resume leaves the manifests/<run id>.json copy consistent with the manifest.json that Prepare rewrites, as Runner.launch does after saving the run record."
  - "internal/shuttleengine/relaunch.go, removeGoneStrand and its caller: settle whether at every point of the resume the saved run record names a strand that still exists, including after a failed AddStrand or saveState and after a kill between steps."
  - "internal/shuttleengine/stepstop.go, StopStepRuns, against attach.go leftoverThenAgeVerdict: settle whether recording stopped for a tied run whose strand is untracked or has a cleared pane binding lets Attach respawn beside an agent that may still be working."
  - "internal/shuttleengine/relaunch.go, relaunchForReprompt and checkResume: settle whether a warning that a live holder of the session could not be ruled out still allows the resume to launch."
  - "internal/shedverbs/loop.go and internal/loomcli/arm.go: settle whether a running shuttle run left by a dead step is reconciled on the re-step before its first child starts when the stop at interrupt time failed or was never reported."
  - "internal/shuttleengine/gate_test.go, TestGate_RepromptToGoneStrand, and internal/shedverbs/loop_test.go: settle which of the resume's branches (tracked dead pane, cleared binding, unready provider, failures at holder lookup, checkResume, Prepare, AddStrand and record save) and which OnStepOrphaned paths (panic with a recorded child, empty note, specFor wiring) have no test."
---

Each entry names a site and the question to settle.
