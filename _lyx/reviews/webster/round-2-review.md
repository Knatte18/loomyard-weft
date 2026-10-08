---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/shuttleengine/waitshadow_test.go:62
    summary: no row pins an API-error marker that lands after the turn end was folded, so the shadow's turnEnd re-read at refresh is uncovered
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/waitshadow.go:110
    summary: the shadow re-reads the whole events file on every poll tick and slices from the cursor, where the loop's own reader reads only from its offset
  - id: F3
    severity: LOW
    class: consistency
    location: internal/shuttleengine/doc.go:193
    summary: the provider-neutral package doc names Claude's hook event Notification, while the matching comment in activity.go words the same point neutrally
---

The range is `git diff $(git merge-base main HEAD)..HEAD`.
`_lyx/shed/shuttle-state/status.json` gives `product.parent` as `main`, and `_lyx/loom/rework/` does not exist, so no live rework round applies.
I ran `go build ./...`, `go vet` with the `integration` tag, `lyx loom lint-comments --base <merge-base>`, and `go test` over `cmd/lyx` and the shuttle, batten, orch and webster packages, plus the `integration` tier of `shuttleengine` and `shuttlecli`.
All passed.
No card carries a `Rename` or `Delete` group, so the per-type mechanical checks have nothing to run.
The one retired symbol the cards name, `denyInstalls` (card 1), has no remaining reference.

### [MEDIUM] No test pins an API-error marker that appears after the turn end was folded

**Location:** `internal/shuttleengine/waitshadow_test.go:62`, with `internal/shuttleengine/fakes_test.go` (`sessionFakeEngine`).
**Issue:** `logSessionState` keeps the newest turn start and the newest turn end and re-reads both signals' transcript markers at every refresh (`waitshadow.go:136-148`).
That is what makes the wait loop's fold agree with the whole-file fold when a marker lands after its signal was folded.
The table has a row for the turn-start half: "an interrupt marker that lands after its turn start was folded is read at a later refresh".
Nothing covers the turn-end half.
`sessionFakeEngine` embeds `waitingEngine` and implements no `ActivityReader`, so `readSessionFacts` never reaches `TurnEndActivity` in `Wait`.
`sessionread_test.go`'s `readEngine` covers the API-error marker only in the files-only reader.
Dropping the `shadow.turnEnd` assignment at `waitshadow.go:130-131` would leave every test green.
Under `PATTERN-test-economy` this is a behavior the diff adds with no coverage.
**Fix:** Add a `TurnEndActivity` method to `sessionFakeEngine` (the API-error counterpart of `interrupted`/`interruptAt`), and add one row to `TestWait_LogsSessionStateBesideItsClassification`.
In that row the turn end is folded first with the marker absent, then the marker is set, and a later refresh logs the `idle-stalled/api-error` change.
The row extends the existing function and adds no top-level test, so the `test-budget.yaml` row stays.

### [LOW] The shadow re-reads the whole events file each tick

**Location:** `internal/shuttleengine/waitshadow.go:110-121`
**Issue:** `os.ReadFile(run.state.EventsPath)` loads the entire file on every poll tick, and only `data[shadow.cursor:]` is parsed.
The wait loop's own reader, `readEventsFrom` (`wait.go`), reads from an offset to the last complete line.
Turn-end payloads carry `last_assistant_message`, so the file grows over a long run, and the shadow then reads it all at the poll interval for a log line no decision depends on.
**Fix:** Read from `shadow.cursor` with `readEventsFrom`.
The parser's `consumed` count still advances the cursor, and a trailing stamp is still re-read with its payload.

### [LOW] Neutral package doc names a Claude hook event

**Location:** `internal/shuttleengine/doc.go:193`
**Issue:** The paragraph ends "so a Notification, prompt submission or stamp append never moves it".
`Notification` is Claude Code's hook event name, and `PATTERN-shuttle-provider-seam` keeps provider specifics under `claudeengine`.
The comment on `eventsFileActivity` in `activity.go:122` says the same thing neutrally: "an idle notice, a prompt submission or a stamp line".
The rest of the neutral layer words these lines neutrally as well (`SessionSignalIdleNotice`).
The `Stop` and ask vocabulary the doc also uses is long established in this package, so only this term is new.
**Fix:** Word it as in `activity.go:122`: "so an idle notice, a prompt submission or a stamp line never moves it".

## Focus verdicts

1. **Marker re-read in `logSessionState`:** not a defect.
   The refresh passes `readSessionFacts` the newest turn start and turn end kept across ticks, not only the signals read since the cursor.
   `Wait` sets `refreshFacts` on every liveness tick (`wait.go:284`), so a later tick with no new bytes re-reads both markers.
   The kept signals are stripped of their session id, and the fact read probes `shadow.sessionID` instead.
   Both marker reads use `Raw` only, so stripping the id does not change them.
   The wait loop's final fold therefore matches the whole-file fold in `readRunSessionState`, as does the `OutputsExist` it feeds, since `Spec.Validate` requires at least one output file.
   `waitshadow_test.go` pins the turn-start marker; the turn-end marker is not pinned (F1).
2. **Comment citations:** no violation found.
   Added comments in `claudeengine/signals.go`, `internal/shuttlecli/smoke_turns_test.go` and the rest of the diff cite `ParseSessionSignals`, `SessionSignalParser` and `PATTERN-shuttle-provider-seam` as stable identifiers.
   I found no card number, decision label or plan reference in an added Go comment.
3. **`session_test.go` parallelism:** `TestRunner_SessionState` calls `t.Parallel()`.
   `TestWait_LogsSessionStateBesideItsClassification` is serial and its doc comment names the process-wide log capture, as `PATTERN-test-speed` requires.
