# Plan-Burler round 1 — fixer report

All five findings were validated against the code and the decision record, and all five are fixed.
`lyx loom validate-plan` passes after the fixes; its only findings are the informational `create-new-unit` notes for card 1's three new corpus files.

## Changes

- **F1 (MEDIUM, consistency) — fixed as proposed.**
  Premise checked: `internal/shuttleengine/claudeengine/testdata/corpus/<case>/` holds `case.yaml`, `events.jsonl` and `transcript.jsonl`, and `TestCorpus_ReplaysEachTurnEnd` replays each through `ParseEvents`, the wait loop and the fold, checking `states:`; D1 says the lines "are trimmed into `claudeengine` corpus cases".
  Card 1 now creates `testdata/corpus/hook-capture/{case.yaml,events.jsonl,transcript.jsonl}` in place of the two `testdata/hooks/` files, with the gate and registry ordering in `case.yaml` free fields beside the existing cases' `source`, `date` and `claude_code_version`, and `states` holding today's fold readings.
  The scenario drives one run, with a resumed launch of the same run that is killed, so every hook line lands in the case's one events file.
  `corpus_test.go` joins no card: `loadCorpusCase` decodes with `yaml.Unmarshal`, which ignores the free fields.
  Card 2 (new) points its `TestParseSessionSignals` source rows at the case's `events.jsonl` and `Uses:` it; card 3 edits the case's `case.yaml` so the `startup`, `clear` and `resume` lines read idle-fresh; card 4 edits it so the `PermissionRequest` line's asking entry joins `states`, and points its parser rows at the case.
- **F2 (MEDIUM, scope) — fixed as proposed.**
  Premise checked: `engine.go`'s `InputBoxClearer` doc says "The caller sends the clear only into a box it has just read as holding exactly its own text", and `session.go`'s `Claude.ClearInputSequence` doc rests on "Sent text is one line with the caret at its end".
  Card 10 (formerly 8) adds `internal/shuttleengine#InputBoxClearer` and `internal/shuttleengine/claudeengine#Claude.ClearInputSequence` to `Edit:`, and its Intent says both docs name the two boxes the clear goes into.
- **F3 (LOW, decision) — fixed as proposed.**
  Premise checked: `awaitIdleSession` in `send.go` loops over `busyReading` until its window or poll cap, so a held post-clear reading is followed by more readings in the same attempt.
  Card 10 now says every later reading of the same attempt is a post-clear reading: a full guard reading that reads exactly the recorded character as residue and never clears again; any other box, including a different lone candidate, holds as a draft.
  I chose this over holding the recorded character as a draft because the draft reading would bring back #554's stall for the rest of the attempt, which D8's rationale removes, while every such reading still takes D8's full guard.
  `TestSend_WaitsForIdleSession` gains a row: operator activity at the post-clear reading, then a quiet operator at the next reading with `+` still on screen, sends with no second clear key.
- **F4 (LOW, scope) — fixed as proposed.**
  Premise checked: `buildSettings`'s `recordingHook` takes `alwaysSucceeds`, and card 1 named neither it nor the hooks' output.
  Card 1's Intent now says both new recordings use `recordingCommand` with `alwaysSucceeds` true, exit 0 and print nothing, so they answer no permission request and add nothing to the session's context; the `TestBuildSettings_RecordingHooks` rows assert the `; true` suffix, as the existing rows at `settings_test.go:102`–`105` do.
  The ImpactSummary says the entries are silent and exit 0.
- **F5 (LOW, design) — fixed as proposed, with `lyx loom insert-card`.**
  New card 2 `session-start-source` creates `SessionStartSource`, its four constants and `claudeengine#sessionStartSource`, re-types `SessionSignal.Source`, edits `signalFromPayload`, and carries the source rows of `TestParseSessionSignals`.
  The fold card, now card 3, keeps the fold, readiness and idle-enumeration changes and `Uses:` the four source constants.
  New card 9 `stray-input-reader` creates `StrayInputReader` and `Claude.StrayInput` with the candidate table and the `test-budget.yaml` row.
  The guard card, now card 10, keeps the guard, the confirm path and the PATTERN background edit, and `Uses:` `StrayInputReader`.
  Prose that names a moved card was updated by hand: the overview's conditional card (now 7), card 1's blocker clause (card 7) and ImpactSummary (card 4), and card 6's fake-engine sentence (card 3).
  The Card Index line of card 3 now describes the fold only.

## Disputed

None.

## Deferred

None.
