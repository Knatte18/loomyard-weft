---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/webstercli/run.go:63
    summary: The lyx webster run help still says it clears any leftover pause flag, but Run no longer clears one
    fix: In the run verb's Long help text in internal/webstercli/run.go, change "by hand), clears\nany leftover pause flag once those refusal gates pass, archives any stale\noutcome.yaml/summary.md," so the sentence reads "by hand), archives any stale\noutcome.yaml/summary.md,", dropping the clause about the pause flag.
  - id: F2
    severity: LOW
    class: scope
    location: internal/websterengine/outcome.go:4
    summary: Comments in webster files still name the deleted pause.go, the pause clear or the pause verb
    fix: "Edit these comments, leaving code unchanged. internal/websterengine/outcome.go:4 drop pause.go from the list \"gitwrap.go/fingerprint.go/pause.go/archive.go\". internal/websterengine/runlevel.go:6 drop \"the never-instantly-re-pause clear,\" from the file header. internal/webstercli/cli.go:69 and internal/webstercli/wiring.go:297 drop \"pause\" from the list of spawn-free verbs. internal/webstercli/cli_test.go:2 and :116 drop \"pause\" from the verb lists. internal/webstercli/sync.go:5 change \"both round-loop modules' pause flags\" to name the one remaining round-loop pause flag, treadle's."
---

### [LOW] run help promises a pause-flag clear that was removed

**Location:** internal/webstercli/run.go:63
**Issue:** Card 28 deletes `ClearPause` from `Run`, so the run verb no longer clears a pause flag. The CLI help for `lyx webster run` still lists that step in its sequence of actions. Card 28's intent is that no reference to the retired flag remains in CLI help.

### [LOW] Stale comments naming the retired pause file, clear and verb

**Location:** internal/websterengine/outcome.go:4, internal/websterengine/runlevel.go:6, internal/webstercli/cli.go:69, internal/webstercli/wiring.go:297, internal/webstercli/cli_test.go:2, internal/webstercli/sync.go:5
**Issue:** `pause.go`, `ClearPause` and `pauseCmd` are deleted, but these comments still describe them as present. A reader following `outcome.go`'s header looks for a `pause.go` that no longer exists. The card's Intent lists only some of these edits, so the card text did not catch them.

## Checked and found sound

- Card 28: no `ErrPaused`, `outcomePaused`, `websterOutcomePaused`, `RequestPause`, `PauseRequested` or `ClearPause` for webster remains. `parseOutcome` accepts `done` and `stuck` only, and the docs, Master stencil and `refusal-spec.md` no longer name `paused`. Every Delete target listed in the card is gone. The `treadleengine` pause flag and its tests are a different module's and stay.
- Cards 5 and 6: `runCore` calls `pauserecord.Network` for every subcommand in `remoteSubcommands` (push, fetch, pull, ls-remote, clone) before the deadline context starts. `authRT.RoundTrip` holds every GitHub request, and `NewWithBaseURL` always installs it. `Network` returns a no-op when no gate is installed.
- Cards 8, 16 and 19: batten's two idle clocks, the step watchdog and shuttle's deadline all measure time through `pauserecord.AwakeSince` or `Excluded` over the sleeps and the coverage's intervals. The old running total (`sleepGap`, `asleep`, `asleepStamp`, `clocksSeen`) is gone from production code and tests.
- Cards 7, 14, 15 and 27: the parent lock passes over a covered ticket without dropping it. Reed resume returns covered strands as `Held`. `loom start`, `loom resume` and `batten run` refuse a covered slug with the resume way forward. An unreadable record or geometry reads uncovered in every reader (log one Warn and carry on).
- Card 16: its Delete group is absent from the tree.
- Test economy: no new test found that repeats an existing one. The batten and loom pause-hold tests cover separate CLIs.
- Run: `lyx gate test` over websterengine, webstercli, shedadapters, contracts/stencils, pauserecord, gitexec and githubclient passes. I did not run the full suite or the integration tier.
