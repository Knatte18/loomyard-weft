```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's changed code through the fix-code-correctness lens. I made no further tool calls. I found no wrong results and no broken callers, but two tests don't pin what they claim to.

- **MEDIUM** — `TestGate_RepromptToGoneStrand`, finding F4 (old strand removed after the record names the new one). Evidence:
  - The only row that removes the old strand is "a dead own-window pane resumes…" (`wantRemoved: ["strand-1"]`).
  - It asserts only the final `RemoveStrandCalls` list and the status queue gives the same call sequence either way, so it passes with the old order (remove before `AddStrand`).
  - The AddStrand-failure and save-failure rows use an untracked old strand, so they can't show whether it was removed.
  - Suggested pin: a row with a removable tracked old strand plus `addStrandErr` (or `saveFails`), asserting the old strand is not removed.

- **MEDIUM** — F2 (`SkipSkillLoadTurn`). Evidence:
  - The gate test checks only that `Prepare` received the skills and the flag.
  - The engine behaviour is untested: `claudeengine.go` `loadsSkills` and the `codexengine.go` `!spec.SkipSkillLoadTurn` guard.
  - Reverting either guard would pass every test, though the review's premise was the engines' settings and launch line.
  - Suggested pin: an engine `Prepare` test with `Skills` and `SkipSkillLoadTurn` set, asserting `Launch.PromptLine == ""` and that the settings still carry the skills' scope.

- **LOW** — F7 (`newestInterruptedTraceID` in `loop.go`). Evidence:
  - It picks the newest `kind: interrupted` JSON in `StepsDir`, by mtime.
  - That includes the loop envelope files and any older interrupted record.
  - So a failed status from a non-interruption (a producer error) re-stops the runs of an older interrupted step.
  - It's idempotent, because `StopStepRuns` touches only running runs of that tie, but the selection isn't tied to the failed stop. The test's single record doesn't exercise it.

Traced and held: `relaunchForReprompt` on a failure of each call.
- Offset: `startupEventsOffset` is taken before `Prepare`, and its only readers are the two `wait.go` helpers.
- State restore: `run.state` is restored on a save failure.
- Old-strand removal: it runs after the save and only warns on failure.
- `StopStepRuns`: held and finished branches behave as the added rows expect.
- Defect check: the F1, F3, F6, F9, F10 and F12 tests would fail with their defects in place.

Tool calls: none.
