---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-dialog-notice.md:299
    summary: Card 7 prescribes two guard branches, an engine lacking InputBoxReader or SessionSignalParser and a pane too short to draw the box, that no TestWait_DialogNotice row reaches
    fix: In 07-dialog-notice.md, add two rows to the TestWait_DialogNotice list. First, an ASK line run on sessionFakeEngine, which parses session signals but has no InputBoxReader, with the clock past permissionNoticeBound sends no notice. Second, an ASK line run on readinessEngine with tooShort set and a pane showing no caret, with the clock past permissionNoticeBound sends no notice. Name both inputs in the watchDialog paragraph next to the branches they reach.
  - id: F2
    severity: LOW
    class: decision
    location: _lyx/plan/08-fork-ask-live-scenario.md:356
    summary: Card 8 calls rm -f ./smoke-ask-$$ only the first candidate, so it never settles which command the live scenario runs
    fix: In 08-fork-ask-live-scenario.md, replace the first-candidate sentence with one that fixes the command as rm -f ./smoke-ask-$$. State that it is a computed-path rm through a special parameter card 4 lets through, which the decision record assumes Claude Code still prompts on in bypass mode. State that a run recording no ask fails the scenario, which is the open-risk outcome, and is never retried with another command.
---

### [MEDIUM] Card 7's no-reader and pane-too-short guards have no test that reaches them

**Location:** `_lyx/plan/07-dialog-notice.md:299` (the `watchDialog` paragraph) and the `TestWait_DialogNotice` rows at lines 323–334.

**Issue:** The Intent says `watchDialog` does nothing for an engine that lacks `SessionSignalParser` or `InputBoxReader`.
It also says a pane too short to draw the box is a failed read that changes nothing.
The test rows cover an interactive run, a runner without a notifier and a failed capture.
No row runs an engine without the input-box reader, and no row runs a too-short pane.
The no-reader guard is the one that matters most.
`readBox` (`internal/shuttleengine/readiness.go:49`) returns `boxReading{}` for an engine without `InputBoxReader`, so `read` is false.
That is the same reading a real "no input box" pane gives.
If the guard is ever lost, a provider without a box reader sends a false dialog notice on every ask.
No test would catch it.
The fixtures this needs already exist in `internal/shuttleengine/fakes_test.go`.
`sessionFakeEngine` has the parser and no `InputBoxReader`.
`readinessEngine` has a `tooShort` field.
The decision record (D6) lists "an engine without session signals or an input-box reader send nothing" as required behavior, and the rubric's reachability rule asks for a named test input for each prescribed guard branch.

### [LOW] Card 8 leaves the live scenario's command open

**Location:** `_lyx/plan/08-fork-ask-live-scenario.md:356`.

**Issue:** The card says "the first candidate is a computed-path `rm` through a special parameter card 4 does not list, such as `rm -f ./smoke-ask-$$`".
"First candidate" and "such as" let the implementer try other commands until one records an ask.
That would hide the decision record's open risk, which says that if no ask is recorded, D6's latch needs another trigger.
The command itself is consistent with the rest of the plan.
`$$` passes card 4's `[$][A-Za-z0-9_{@*(]` class, because the first `$` is followed by `$` and the second by the JSON closing quote.
It passes card 3, which only matches `sed`/`perl`.
The decision record's auto-mode assumptions say a special parameter D3 lets through still stalls the run.
So the plan can fix this one command and let a failure mean what the open risk says it means.

## Rejected

Focus items I reached and judged not to be defects:

- **Cards 3 and 4, test inputs vs `commandPositionPattern`.**
  Every named deny input reaches the new rows.
  The `&&`, `;` and `|` cases sit after a separator in the head's `[;&|(` + backtick class.
  `/usr/bin/sed` passes through the optional path prefix.
  `sed -n 3p f -i` holds `-i` before the next separator.
  The named admit inputs miss the new rows.
  `--quiet`, `--silent`, `--expression=` and `--file=` open with `--`, so the single-dash cluster rule skips them, and none starts with `--i`.
  In the newline case, the JSON `\n` ends the `sed` words, as `goArgumentsPattern` already does.
  `preToolUseOutput` (`hookcmd_integration_test.go:38`) encodes only `hook_event_name`, `tool_input.command` and `tool_name`.
  So no other payload field holds `-i` or `$`, and card 4's `rm a b` and `rm -f .scratch/a` stay clean.
  No upstream check sees these payloads first: each standing deny is its own PreToolUse hook.
- **Card 7 against D6.**
  The Intent carries every D6 behavior:
  - the latch sets on an ask, and a later ask keeps its stamp;
  - an input box clears it at any age;
  - a failed read changes nothing and is retried;
  - the dialog offset is persisted before the notifier call;
  - the offset advances for an ask folded in after the notice;
  - a reattach rebuilds the latch from the cursor at max(`PromptOffset`, `DialogNotifiedOffset`);
  - `QuietHold` and `HoldDeferral` neither suppress nor defer the notice.

  Every listed `TestWait_DialogNotice` row is reachable through `readinessEngine` and the `ASK` lines `sessionFakeEngine.ParseSessionSignals` reads (`fakes_test.go:649`), apart from the gaps in F1.
- **Card 7 granularity and test economy.**
  The latch, the notice, the `RunState` field and the docs are one reviewable feature.
  The new `TestParseSessionSignals` row runs the same parser branch as `permission_request_is_an_ask_without_an_event` (`signals.go:167`, which never reads `agent_id`).
  It still pins the fork-shaped half of D6's first acceptance criterion at the only layer that reads real payloads, which `sessionFakeEngine` cannot express.
  `TestWait_HeldTurnEndNotice` covers no dialog notice, so a new test for that behavior meets `PATTERN-test-economy`.
  `fakeReed`'s `CaptureErr` fails every capture, and `idleReed` fails every capture from `failAt` on, so a fake whose capture fails on one tick and then recovers is new.
- **Card 8's `Edit` list, and the header comments cards 3, 7 and 8 rewrite.**
  Each header lives in a file inside the package unit of a target the card already names: the `plan:internal/shuttlecli#…` and `plan:internal/shuttleengine/claudeengine#…` handles, and `internal/shuttleengine#Run.Wait`.
  Listing `wait.go` beside `Run.Wait` would trip `redundant-file-target`.
- **Card order and `## verify:`.**
  Card 5 reads no symbol of card 3's deny table.
  Card 8 uses only existing `smoke_turns_test.go` helpers (`newTurnHub`, `startTurnScenario`, `awaitSignal`) and nothing from card 7.
  The section runs the template's order exactly, and `./...` covers `cmd/lyx` and every targeted package, integration-tagged ones included.
- **Edit glyphs and re-sign arrows.**
  No card changes a listed member's parameters, results or receiver.
  Every symbol an Intent changes is listed as a member glyph, apart from derived constants like `noticePythonDeny`, which change with no edit of their own.
