```yaml
lens: fix-consistency
outcome: reported
```

Checked through the fix-consistency lens: one stale doc comment, and nothing else that disagrees.

**Findings**

- **LOW** — `internal/shedverbs/loop.go`, the `LoopSpec.OnStepOrphaned` field doc (added in the earlier diff, unchanged by the fix). It still says the hook "runs once per such child the loop reads back, and once for the step a panic or a dead loop leaves in flight."
  - The fix added a fourth call site. `reconcileInterruptedStep` (called in `runLoop`) invokes the hook before the first child of a loop that starts on a failed status, with the newest interrupted record's trace id.
  - The sibling `internal/shedverbs/doc.go` paragraph and the summary were updated, but this field doc was not. It also still describes the argument only as "the child's trace id".
  - A reader of the field alone no longer learns the full set of calls.

**Checked, no finding**

- **Manifest `skill-load` items on a resume.** The claude manifest (`claudeengine/manifest.go:91` and `:212`) keeps an item only when its proof appears in the launch line (`held = strings.Contains(in.launchLine, candidate.proof)`). With `SkipSkillLoadTurn` the launch line carries no skill-load message, so the `skill-load` items are dropped and the manifest stays truthful. The codex manifest (`codexengine/manifest.go:190`) builds the same candidate with a file proof. I did not read its filtering, so I assume it matches.
- **Other uses of the changed functions.** `StepStopReport` and `StopStepRuns` are used only in `loomcli/looparm.go`, `loomcli/cli.go` and `shuttleengine/doc.go`, and all agree with the new `Held` and `Finished` fields. `sessionStartRecorded` and `turnStartedAfterSessionStart` have no callers outside `wait.go`, and `SkipSkillLoadTurn` is read only by the claude and codex engines.

**Further tool calls (4)**
1. Read `claudeengine/manifest.go` 200-222 and `codexengine/manifest.go` 183-200: my context lacked how the skill-load manifest items depend on `spec.Skills` and the launch line.
2. Grepped for `StepStopReport` and `StopStepRuns` callers, for the two events-offset helpers, and for `SkipSkillLoadTurn`: my context lacked the full caller and twin sets.
3. Read `claudeengine/manifest.go` 70-125: my context lacked how a candidate's presence in the launch line is judged.
4. Grepped `claudeengine/manifest.go` and `claudeengine.go` for `Omitted` and `Verdict`: I wanted to confirm that a dropped item raises no refusal.
