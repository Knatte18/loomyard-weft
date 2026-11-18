```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F26, F27, F28) by building concrete cases where they go wrong.

**[MEDIUM] F27: the bound's live-strand clear can strand a fresh fallback's task prompt**
- Changed text: D7 step 4, "Once the pane is started, any `PromptLine` is sent as `launch` sends it; then the mark clears back to running", together with the bound sentence "one that reads it over a live strand clears the mark to running and attaches, the record already naming the new launch".
- Case: a resume is refused for no transcript, so steps 2 to 4 run again with a fresh launch, whose `PromptLine` carries the task prompt. Step 4 writes the record and reed relaunches the pane. The `Wait` process is then killed before the `PromptLine` is sent.
- Past the bound, an attach finds a live strand, clears the mark and attaches. The pane sits at an empty prompt, because nothing sent the task prompt. The record names the new launch, but nothing marks the `PromptLine` as unsent. The attach then waits until the run's own timeout.
- The proposal only covers the resume case, whose `PromptLine` is empty. It needs either an unsent-`PromptLine` fact in the record that the clearing attach acts on, or for the fallback case to end as a strand that died.

**[MEDIUM] F28: one running predicate for every reader exposes the limited turn end as current activity**
- Changed text: "Every reader of a run record that tests for a running run reads a relaunching record within its bound as a live running run, through one predicate on the record."
- Case: the activity reader (`internal/shuttleengine/activity.go`) reads a running record's activity past its prompt offset. It keeps `reading.UsageLimit` and `APIError` whenever `runIsLive(rs)` holds.
- Between step 1 and step 4's write, the prompt offset still sits before the limited turn end. So a relaunching record read as live running reports the usage limit as the run's current state, for the whole window.
- The session-state reader (`internal/shuttleengine/sessionread.go`, `readRunSessionState` folds the signals past the prompt offset) reads the same stale tail.
- The proposal's blanket rule therefore gives these consumers a stale limit during the relaunch. The rule needs a carve-out: activity and session-state readers report a relaunching run as relaunching, or with no turn-end facts.

**[LOW] F26: "never reaches ready" does not always end as a strand that died**
- Changed text: step 4's "A pane that never reaches ready ends the run as a strand that died", and the matching new acceptance criterion.
- Case: `awaitStartup` (`internal/shuttleengine/wait.go:775-779`) ends with `abandonStartup(OutcomeTimeout, nil)` when the run's own deadline falls inside the startup window.
- So a relaunch late in a run's time that never reaches ready ends as a timeout, not as died. Both the criterion and the step-4 sentence promise died unconditionally.

**[LOW] F28: the "within its bound" predicate contradicts the unconditional step stop**
- Changed text: the predicate reads a relaunching record as running only "within its bound", yet `StopStepRuns` "stops a relaunching run whatever its pane state".
- `StopStepRuns` first filters on the running test (`stepstop.go:78`). Past the bound, with the `Wait` dead and no attach since, the record still reads relaunching, the predicate reads it as not running, and the step stop skips it.
- The orphan sweep keeps any directory whose record reads relaunching, so that record and directory persist indefinitely. The step-stop sentence should either apply regardless of the bound, or end a past-bound record as a strand that died.

Tool calls:
- Read `internal/shuttleengine/activity.go:95-190` and `internal/shuttleengine/sessionread.go:45-75`. My context named these readers but did not hold their bodies, and I needed them to build the F28 case.
