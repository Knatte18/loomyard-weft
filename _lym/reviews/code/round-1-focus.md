---
round: 1
focus:
  - "Part 1, prompt delivery: in `internal/shuttleengine/hold.go`, `wait.go`, `attach.go` and `send.go`, settle whether a deferred prompt is retyped at a held turn end and on attach only while no turn start carrying it exists, and whether a session that has already received it is never sent it twice."
  - "Part 1, landing test: in `internal/shuttleengine/claudeengine/claudeengine.go` and `command.go`, settle whether the check that a turn start carries the prompt can mistake an operator's turn, the skill-loading first message or a partial match for the prompt, and whether the loom history can still record Darn-Write as done before the prompt landed."
  - "Part 2, open launch: in `internal/shuttleengine/open.go`, `engine.go` and `claudeengine/settings.go`, settle whether a session opened with `lym shuttle open` in a hub gets the tool-boundary claim, the turn-end Stop delivery and the idle waiter, the creel trust sentence in its own directory, and a strand marked `CreelHooks`, and whether an open outside a hub is left unchanged."
  - "Part 2, settings placement: settle whether the settings written for an opened session can overwrite or merge wrongly with existing settings in its directory, and whether a second open of the same role or a failed write leaves a half-written state."
  - "Run path: in `run.go` and `claudeengine/command.go`, settle whether a run session's launch is unchanged by the new open-session hook delivery, with the hook settings built once and not duplicated."
  - "Scope: settle whether `internal/fabricengine/recordswritelock_test.go` and the `docs/overview.md`, `creelengine/doc.go` and `shuttleengine/doc.go` edits are licensed by the Goal, and whether the docs for the changed behavior land in the same diff."
  - "Test economy: compare the new tests in `attach_test.go`, `hold_test.go`, `open_test.go`, `run_test.go`, `command_test.go`, `corpus_test.go` and `hookcmd_integration_test.go` for overlap with each other and with existing tests, and check that each added behavior has coverage."
  - "Change description and commits: if `_lym/landing/summary.md` exists, check each of its claims against the diff, and read the three commit messages for claims the diff does not bear out."
---

The decision record carries only the `## Goal` section, so it is the whole measuring stick; its other sections are empty.
