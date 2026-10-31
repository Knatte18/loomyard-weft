```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running the changed functions and branches step by step.

**Per function or branch, with the case I ran**

1. `runTurnStarts` (`carryover.go:67-92`), prompt detection:
   - Case: `PromptLine` empty, events `[S1:"a", S2:"b"]`.
   - Result: S1 sets `promptSeen` and has no operator instruction. S2 is operator unless its text is in the sent-texts file.
   - This matches the old loop's `continue` and append logic, case for case. Correct.
   - Case: `PromptLine` set to `p`, first turn start has instruction `x`.
   - Result: the first turn start is operator, as before, because the prompt never landed. Correct.
2. `runTurnStarts`, clamped prompt offset:
   - Case: `PromptOffset` is -5 or past `len(data)`.
   - Result: the clamp is the same expression as before. `positionedTurnStarts` gets `data[promptOffset:]` and `startOffset=promptOffset`, so `end` is a true file offset. Correct.
3. `runTurnStarts`, turn start whose `Raw` is empty or not found:
   - Case: a signal with empty `Raw`.
   - Result: `positionedTurnStarts` (`wait.go:972-974`) drops it, where the old loop counted every `SessionSignalTurnStart`. This is a real change in what the carry-over reads (acceptance criterion D1).
   - The only parser in production, `claudeengine/signals.go`, always sets `Raw` from the line it parsed, so nothing changes there. The test fake `instructionEngine` set no `Raw`, which is why the diff edited `carryover_test.go`.
   - If a skipped turn start were the prompt's, `promptSeen` would shift to the next one. Not reachable with the Claude parser.
   - Verdict: correct for production, but undocumented. Reported as F1 (LOW).
4. `runTurnStarts`, unparsable engine or unreadable events file:
   - Result: both return nil, as before. `operatorInstructions` then yields a nil slice. Correct.
5. `positionedTurnStarts` and `turnStartOffsets`:
   - Case: a non-turn-start signal is found.
   - Result: `searchFrom` advances exactly as in the old `turnStartOffsets`, and only turn starts are appended. The offsets returned by `turnStartOffsets` are identical, so the hold-guard callers at `wait.go:867`, `933` and `937` are unaffected. Correct.
6. `operatorStartedTurn` (`hold.go:109-118`), boundary:
   - Case: turn start line followed by turn end line, so `start.end` is at most the turn end's line start, which is below `held.offset`. A turn start after the turn end has `end > held.offset`.
   - Result: `start.end >= held.offset` breaks exactly on starts at or after the turn end. The `>=` equality case needs the turn start and turn end on one line, which cannot happen. Correct.
7. `operatorStartedTurn`, prompt-only run:
   - Case: events `START, STOP`.
   - Result: the only start is the prompt, so its `operatorInstruction` is empty and the answer is false, so it is notified. Correct.
8. `operatorStartedTurn`, no turn start before the turn end:
   - Result: `instruction` stays empty and the answer is false. Correct.
9. `operatorStartedTurn`, operator turn then a no-instruction turn:
   - Case: events `START:op, STOP, START, STOP`.
   - Result: the newest start before the second STOP is the no-instruction one, so it is false and notified. This is the D2 reading. Correct.
10. `operatorStartedTurn`, `held.offset` when `Raw` is not found, or repeated identical STOP lines:
    - Result: `offsetPastEvent` (`wait.go:1003-1013`) uses `LastIndex`, or the batch end. If the same STOP line text occurs twice in a batch, `held.offset` points past the later one. A turn start between the two would then count as "before" the turn end.
    - This is how `held.offset` was already computed, and Claude's turn-end payloads carry distinct content. Not attributable to the diff, so no finding.
11. `wait.go:319` case ordering:
    - Result: it sits after `continued` and the usage-limit case, and before `wakesOnBackground`, `notifyHeld` and the deferral.
    - With `HoldDeferral != nil`, `pendingHold` is not set, so nothing is deferred.
    - `NotifiedOffset` is untouched because `notifyHeld` is never reached.
    - `beginHeldWait` runs before the switch, so the hold, wait mark and deadline are unchanged.
    - A later lym-started held end has `held.offset > NotifiedOffset` and is notified. Correct.
12. `holdNoticePending() && operatorStartedTurn` under a covering pause:
    - Without deferral, `holdNoticePending` is false while covered. The code falls through to `notifyHeld`, which returns on `covered`, so nothing is sent.
    - With deferral and covered, `holdNoticePending` is true and the operator check runs first. Correct.
13. Failure paths:
    - `os.ReadFile` failing in `runTurnStarts` returns nil, so the turn reads as not the operator's and is notified as today. This matches the record's open risk.
    - A missing sent-texts file gives an empty map, so every typed instruction reads as the operator's. This matches the record's bound.
    - `operatorStartedTurn` re-reads the whole events file and the sent-texts file on every held branch hit. That is a cost, not a correctness issue.
14. `run.state` copied by value into `runTurnStarts` without `recordMu`:
    - Other held-path code does the same (`hold.go:133` passes `run.state` by value to `readRunSessionState`). Precedent exists, so no finding.

**Findings**

- F1, LOW, `consistency`, `internal/shuttleengine/carryover.go:80` and `wait.go:972-974`
  - Summary: the refactor makes `operatorInstructions` read only turn starts whose `Raw` line is found, so a turn start with empty or unfound `Raw` is now dropped. The old loop counted every turn start, and `runTurnStarts` and `operatorInstructions` say nothing about this.
  - Evidence: the diff changed the `instructionEngine` fake to set `Raw`, which shows the reading changed. Production Claude always sets `Raw` (`claudeengine/signals.go:126-128`), so no production regression.
  - Fix: either state the requirement ("a turn start whose `Raw` line is not found in the events file is not read") in the `runTurnStarts` doc comment, or accept that no `SessionSignalParser` in the repo yields an empty `Raw`.

No design, BLOCKING or MEDIUM defects found.

**Tool calls**
1. `grep` for `recordMu` and `PromptLine` in `internal/shuttleengine` non-test files, to check whether copying `run.state` unlocked races with locked writers. My context lacked the lock sites.
