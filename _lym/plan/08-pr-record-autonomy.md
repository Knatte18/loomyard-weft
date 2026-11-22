# Card 8 — pr-record-autonomy

**Create:**
- `plan:internal/pullrequest#ClearReviewSession` -> `func ClearReviewSession(recordPath, lockPath string) error`
- `plan:internal/pullrequest#TestClearReviewSession` -> `func TestClearReviewSession(t *testing.T)`

**Edit:**
- `internal/pullrequest#Record`
- `internal/pullrequest#TestRecord_RoundTripAndUpdate`

**Uses:**
- `internal/pullrequest/doc.go`

**Intent:** D4: the pull request record holds the review's autonomy, recorded once per review, and a goto onto PR-Review clears it with the session.

- `Record` gains `Autonomy *bool` (`json:"autonomy,omitempty"`): nil until PR-Review records the run's value, then true or false for the rest of the review.
  A record written before this change reads with no autonomy, so the next session start records it.
- `ClearReviewSession` clears `Autonomy`, `StrandGUID`, `StrandName`, `SessionID` and `Engine` under the record's lock, keeping every other field,
  so a re-entry of PR-Review records the autonomy anew and starts a new session.
  A missing record is no error, since a goto onto PR-Review of a run that never opened one has nothing to clear.

Tests, tier0:

- `TestRecord_RoundTripAndUpdate` asserts a record with `Autonomy` pointing at `false` round-trips as an explicit false, and one with it nil writes no `autonomy` key.
- `TestClearReviewSession`: a record holding autonomy, session fields, `GatedHead`, `Continue` and `SendBacks` keeps the last three and loses the first two;
  a missing record returns nil and creates no file.

**ImpactSummary:** One tolerant optional field and one clearing writer in the record package; no existing writer touches the field, and the record's file shape is unchanged for records without it.
