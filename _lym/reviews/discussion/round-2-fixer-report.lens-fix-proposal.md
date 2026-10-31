```yaml
lens: fix-proposal
outcome: reported
```

Task: as the fix-proposal lens, test each proposal the fixer carried out and build a concrete case where it goes wrong.

Proposals carried out as written: F1 (D6 stops the session on entry, plus the new acceptance case), F2 (D5 sets `DenyPublish` and keeps the existing steer), F3 (D4's scoped "live one" sentence plus the sentence about sessions reopened outside the row), F4 (D4's Bound says "call's one diff").

**MEDIUM**: D4's new sentence: "otherwise its `decision add` is refused until `lym loom resume` re-enters the row and re-records it."
- **Summary:** the way forward does not re-record the session that was reopened outside the row. `lym loom resume` resumes the old session id a second time, so two agents end up on one session.
- **Evidence:** `internal/landingshed/prsession.go:60-79` is the model D5 follows. It decides liveness only by `rec.StrandGUID` (`pr.SessionLive(rec.StrandGUID)`). When that strand is not live, it sets `spec.ResumeSessionID = rec.SessionID` and starts a new strand. Take the "otherwise" case: the session was reopened under a strand other than the recorded one. The row then finds the recorded strand not live and resumes the recorded session id. That session's process is already running under the outside strand. The row records the new strand, but the outside strand stays live and unadmitted. Separately, `internal/reedcli/resume.go:1-3,19-22` shows that `lym reed resume` recreates the persisted strand itself. For that path the "otherwise" branch never happens, so the sentence describes a case only the stop chain could produce, and that case's way forward is the broken one above.

**MEDIUM**: D6's changed entry bullet: "on one carrying the current stamp it first stops the recorded session when its strand is live".
- **Summary:** the stop is keyed on the recorded strand. A session the F3 sentence now admits can exist, one reopened while halted under a different strand, is not stopped. The resume then ends the row Done with that agent still live. This is the hole F1 set out to close, and its acceptance case does not cover it.
- **Evidence:** in `prsession.go:172-185`, `stopLiveSession` checks and stops only `rec.StrandGUID`. Concrete sequence: the run halts at the row, the stop chain reopens the session under strand B while the recorded strand is A, `sampler-done` runs on a clean checkout, and `lym loom resume` enters the row. A is not live, so nothing is stopped and the row ends Done. B can still commit or dirty the checkout during Describe and Finalize, which breaks D6's Bound: "Describe and Finalize always start from a committed tree". The new acceptance clause only covers a reopen by `lym reed resume`, which keeps strand A.

F2 and F4: no findings. `prsession.go:147` sets `DenyPublish: true` for PR-Review's session, so the proposal matches the model D5 follows. F4's wording matches what the review quotes from the `internal/loomcli/decision.go` package doc.

Further tool calls:
1. Grepped `internal/landingshed/prsession.go` for liveness, stop and `DenyPublish`, and listed `internal/reedcli`. My context did not show how the session model decides liveness and stops, or where reed resume lives.
2. Grepped `internal/reedcli/resume.go`. My context did not show whether `lym reed resume` keeps the strand.
3. Read lines 1-60 of `internal/reedcli/resume.go`. My context lacked reed resume's replay contract (same persisted strand or a new one).
