# Card 1 — shuttle-session-id

**Create:**
- `plan:internal/shuttleengine#newestSessionID` -> `func newestSessionID(rs RunState, signals []SessionSignal) string`
- `plan:internal/shuttleengine#Run.SessionID` -> `func (run *Run) SessionID() string`

**Edit:**
- `internal/shuttleengine#RunSessionState`
- `internal/shuttleengine#readRunSessionState`
- `internal/shuttleengine#readSessionFacts`
- `internal/shuttleengine#TestReadSessionStates`
- `internal/shuttleengine#TestRunner_Start_HappyPath`

**Intent:** Expose the provider session id engine-neutrally (D5), additive to shuttle's read-only session reading, and return it from a started run so orch's start seam can record it (D4).

- `RunSessionState` gains a `SessionID string` field, documented as the session id of the newest signal that names one, else the run record's; empty when neither names one.
- `newestSessionID` holds the pick `readSessionFacts` makes today: walk `signals` from the newest back, take the first non-empty `SessionSignal.SessionID`, else `rs.SessionID`.
  `readSessionFacts` calls it in place of its own `sessionID` tracking in the loop, and keeps using the result for `ProcessLiveness`; its loop keeps finding the newest turn end and turn start.
- `readRunSessionState` sets `reading.SessionID` to `newestSessionID(rs, signals)` on the parser path, and to `rs.SessionID` on the no-parser path that reads unknown unsupported, so a run whose engine parses no signals still reports the id it launched with.
  The id is therefore folded over every signal past the prompt offset, never one tick's increment.
- `Run.SessionID` returns `run.state.SessionID`, the id the engine's launch named: minted for a fresh launch, `Spec.ResumeSessionID` for a resume.
  Its doc says it is available as soon as `Start` returns, as `StrandGUID` is.

Tests:

- `TestReadSessionStates` gains a `wantSessionID string` column, asserted against `got[0].SessionID` whenever a reading is returned.
  Every existing row's value is its newest id-naming signal's id, else its record's `sessionID`, else empty: the rows on `finished`, `startLine`/`endLine` and `promptOffset` with no record id want empty;
  "liveness is looked up by the newest signal's session id, not the record's" wants `newer`, which pins a newer signal's id replacing the record's;
  "a dead process reads dead" (record `recorded`, `startLine` naming no id) wants `recorded`, which pins the fallback to the record's id.
- A new row "a later signal naming no id keeps the newest id named": record `launch`, events `END@2026-10-08T07:05:00Z|post-clear` then `START@2026-10-08T07:20:00Z` with no id, no output file; want `{"hub:task:driver", SessionBusy, SessionCauseTurn, 07:20}` and `wantSessionID` `post-clear`.
  It pins that a later tick reading no new id keeps the post-clear id.
- The "an engine without a signal parser reads unknown unsupported" row gets its own `readRun` literal, `finished`'s fields plus `sessionID: "recorded"`, leaving the shared `finished` unchanged, and wants `recorded`.
- `TestRunner_Start_HappyPath` asserts `run.SessionID()` equals `session-1` in place of reading `run.state.SessionID`.

**ImpactSummary:** Additive: one new field on `RunSessionState` and one accessor on `*Run`; `readSessionFacts`'s liveness lookup keeps the same id, and no caller outside `internal/shuttleengine` changes.
