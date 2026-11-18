```yaml
lens: fix-way-forward
outcome: reported
```

I checked the ways forward the fix introduced, as the fix-way-forward lens.

No findings.

- **Clearing the mark over a live strand past the bound:** this leads somewhere. Step 4 now writes `PromptLine` and the prompt offset before reed relaunches the pane. So an attach that clears the mark reaches the existing `DeliverPendingPrompt` (`internal/shuttleengine/run.go:900`), which types a fresh-launch fallback's unsent `PromptLine` past the recorded offset. The cleared started flag sends that attach through the startup probe that already runs with the events proof off.
- **A pane that never reaches ready:** it ends as a strand that died through `awaitStartup`'s `abandonStartup` (`internal/shuttleengine/wait.go:771`). That path is reachable.
- **A stop between step 4's record write and reed's relaunch:** the stop removes the held strand, reed's relaunch then fails, and that falls under "a failure in steps 2 to 4 ends the run as a strand that died". The died-end write leaves the stop's terminal record as it is, so the stop wins.
- **`StopStepRuns` on a relaunching run:** the held strand is still tracked, so the exemption on pane state is enough to reach `stopRecorded`.
- **The new acceptance criteria:** each one can be reached from the state it names.

Calls:
- grep of `PromptLine` in `attach.go`, `rundir.go` and `run.go`: my context did not show whether an attach delivers a pending prompt line.
- Read of `internal/shuttleengine/run.go:890-940` (`DeliverPendingPrompt`): my context lacked its body.
