---
round: 1
focus:
  - "internal/websterengine/runlevel.go, Run around the Master spawn: trace whether every note path any Master of this run was given ends up in State.MasterNotePaths, including when a spawn, prompt send or save fails or is interrupted after the prompt names the note, and when a later Run takes an early exit or replaces the State."
  - "State.MasterNotePaths across the run's lifecycle: check each path that builds, resets, rebaselines, reloads or re-saves State (runlevel.go, internal/webstercli, accept-audit) and settle whether the list is kept or cleared where the amended board entry's 'this run' requires."
  - "internal/websterengine/state.go, the masterNotePath to masterNotePaths key change: settle what happens to a state.json written by one binary and read by another, in both directions, judged against pattern/PATTERN-tolerant-records.md, and whether the change description's Merge danger section names what this can break."
  - "Every caller of CheckParent and every other place that judges Master's writes (record-batch, run-exit audit, accept-audit): settle whether all of them treat the earlier Master's note the same way, including the case where the resumed Run has friction notes turned off."
  - "Text describing the allowance as one note (audit.go comments on ClassParentWrite and the violation Detail string, the comment near the Master spec in runlevel.go, doc.go, contracts/stencils/webster/webster-template-master.md, contracts/specs): settle for each whether it is still accurate after the change."
  - "The new and changed tests (the resumed row of TestRun_DoneOutcome with its shared resumedNoteState, TestCheckParent, the record-batch row): settle whether each would fail without the fix and checks the behaviour its name states, and whether any duplicates another's coverage."
  - "_lym/landing/summary.md: check each claim against the diff, including that the record-batch audit was affected the same way as the run-exit audit and that the existing accept route clears the finding for a pre-change state."
---
Seed focus for round 1.
The answer key is a decision record alone, and its board entry reports a bug,
so the rubric's Root-Cause trailer check applies to the range.
