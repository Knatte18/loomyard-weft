```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change the diff makes, I looked for everything that calls, reads, asserts or documents it, and checked whether the diff leaves any of it wrong.

**Per change, the set found and whether it is covered**

1. **`liveStrand` errors now wrap `errStrandGone` (`run.go:1522-1531`)**
   - Callers that read the message: `interruptEngine`, the send-by-guid path, `session.go:266/280`, `send.go:205`, and the `notShuttleOrLiveStrandError` wrapper. All are covered, because the message text is unchanged.
   - Callers that test the error with `errors.Is`: only `handleGatedBoundary`. Covered.
2. **`handleGatedBoundary` resumes through `relaunchForReprompt` (`wait.go:610-622`)**
   - Readers of `EndedEarly`: `shedadapters/gateattempts.go:62` and `websterengine/verifygate.go:560` both render "never re-prompted, since %s". That wording still reads correctly with the combined error. Covered.
   - The `handleGatedBoundary` comment and the package docs (`wait.go` header, `doc.go:777-782`) were updated. Covered.
   - The `checkLivenessTick` doc and its `eventsProof` invariant (`wait.go:1469-1473`) were not updated, and the resume now breaks that invariant. See finding 1.
   - The rule in the `PATTERN-shuttle-provider-seam` background (lines 78-93) that every removal of a run's strand goes through the stop verb: `removeGoneStrand` and the cleanup after a failed save call `reed.RemoveStrand` directly. Both are inside `shuttleengine`, and `launch` and `abandonStartup` already remove strands the same way. The removed strand is replaced and the record is rewritten right after, so a record never names a removed strand while the run is live. I judged this covered and raise no finding.
   - `claudeengine.Prepare` writes `spec.Prompt` into `&lt;runDir&gt;/prompt.md` (`claudeengine.go:196,223`) and rewrites the launch manifest. The resume therefore overwrites the run's original prompt file, which `RunState.PromptPath` names. See finding 3.
3. **`StopStepRuns` now settles runs of the step whose strand is dead or untracked (`stepstop.go:45-97`)**
   - Callers: `loomcli.stopStepRuns`, through both `watchdogKillNote` and `orphanedStepNote`. Covered.
   - Docs: `doc.go:396-401` and the function's own doc. Covered.
   - Test: `stop_test.go`. Covered.
   - Readers of the record it writes: `attach.go` `dispositionCandidate` and `leftoverThenAgeVerdict` (`attach.go:512-610`). These are not covered. See finding 2.
4. **New `LoopSpec.OnStepOrphaned` (`loop.go`, `loopwait.go`)**
   - Wiring: `arm.go:445`. Covered.
   - Docs: `shedverbs/doc.go` at the three sites. Covered.
   - Tests: `loop_test.go` and `loopwait_test.go`. Covered.
   - `stepRunsNote` is now shared by `watchdogKillNote` and `orphanedStepNote` (`looparm.go`), and `looparm_test.go` covers both. Covered.
5. **The `strandAddSpec` extraction (`run.go`)**: its callers are `launch` and `relaunchForReprompt`. Covered.
6. **Commit messages, `Pattern-Departure:` trailers and declared PATTERN changes**: the range has no trailers, and this round declares no PATTERN changes. Nothing is owed.
7. **Change description (`_lym/landing/summary.md`)**: it claims "the package has always refused to respawn over an untracked strand". See finding 4.

**Findings**

1. **MEDIUM, class `design`**, at `internal/shuttleengine/relaunch.go:262` (the `awaitStartup` call), against `internal/shuttleengine/wait.go:1469-1473` and `wait.go:1516`.
   - Summary: the resume reuses the run's existing events file but runs `awaitStartup`, which passes `eventsProof=true`. The doc at `wait.go:1471-1473` says only the launch's own startup wait sets that flag, "since its events file is created fresh for the launch, so only this launch's hooks satisfy it". That is no longer true.
   - Effect: the first incarnation's session-start and turn-start lines satisfy `turnStartedAfterSessionStart` on the first tick. The resumed run is marked started without its pane ever being probed, so a startup gate such as a trust prompt is never dismissed.
   - The `checkLivenessTick` doc is left wrong. `TestGate_RepromptToGoneStrand` seeds only `STOP:turn1`, so it never reaches this case.
2. **MEDIUM, class `design`**, at `internal/shuttleengine/stepstop.go:91-96`, against `internal/shuttleengine/attach.go:556-566` and `attach.go:597-609`.
   - Summary: `StopStepRuns` now writes `stopped` into a running record whose strand is untracked or has a cleared pane binding. Its doc's reason is "no process waits on it", but the agent pane lives in tmux, outside the killed step tree.
   - Attach's verdict for exactly those two cases (`leftoverThenAgeVerdict`) treats a terminal outcome as respawn-eligible at any age. Before this change, a running record there was held to the `minAge` guard, and a younger one was kept as `verdictError` because a live agent could not be ruled out.
   - Settling the record therefore skips the duplicate-agent guard the attach probe enforces. The attach code that reads this record is not adjusted, and the `stepstop` doc does not mention the case.
3. **LOW, class `consistency`**, at `internal/shuttleengine/relaunch.go:232`, against `internal/shuttleengine/claudeengine/claudeengine.go:196` and `claudeengine.go:223`.
   - Summary: `Prepare` is called on the same run directory, so it overwrites `prompt.md` with the re-prompt text and rewrites the run's launch manifest. After a resume, `RunState.PromptPath` names the re-prompt instead of the run's original prompt.
   - I did not search for readers of `PromptPath`, so how much this matters is unconfirmed.
4. **LOW, class `consistency`**, at `_lym/landing/summary.md`, in the paragraph "The package has always refused to respawn over an untracked strand", against `internal/shuttleengine/attach.go:568-584` and `attach.go:604-606`.
   - Summary: the claim is not borne out. `dispositionCandidate` sends an untracked strand to `leftoverThenAgeVerdict`, which returns respawn-eligible once the run directory is older than `minAge` (the age escape the code comment describes).

**Tool calls**

1. Grepped `.md`, `.go` and `.tmpl` files for `OnWatchdogKill`, `StopStepRuns`, `Untied` and the gate-end messages, to find docs and consumers outside the manifest.
2. Grepped for `eventsProof`, the respawn wording and "untracked strand", to find invariants the resume touches.
3. Read `doc.go:436-470` and grepped `attach.go` for "respawn", to find the readers of a record `StopStepRuns` now settles.
4. Read `attach.go:495-620`, the Attach verdict rules, which the manifest did not include.
5. Grepped `pattern/PATTERN-shuttle-provider-seam.md` for the stop, resume and respawn rules, since the background file was not in the manifest.
6. Read `PATTERN-shuttle-provider-seam.md:74-126`, the stop-verb and completion-signal sections.
