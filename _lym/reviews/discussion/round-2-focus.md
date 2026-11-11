---
round: 2
exclude_lenses: []
focus:
  - "Decision D2 of _lym/discussion/decision-record.md: does a rewritten decision record, whether after a Bouncer clear, a goto back to Discussion-Write or a resume on an unsettled perch, always reach the parent ask, given that round numbers restart at 1 when a new generation starts (startNextGeneration in internal/shedadapters/amendment.go) and a goto or resume starts no generation (internal/shedadapters/bouncer.go)?"
  - "Decisions D3, D4 and D5: can an ask or answer be matched to the wrong round, receiver or lifecycle state (clear re-seeding round 1 under the same generation via entryClear in internal/shedadapters/bouncer.go, NotifyParent in internal/orchcli/notify.go resolving the receiver by liveness, a chair ending without running the await)?"
  - "Decision D5: is the deadline cap stated with a formula that matches CapAssistTimeout in internal/loomengine/review.go, with a stated outcome when no room is left, rather than left for Plan to find out?"
  - "Decisions D4, D6 and D7: is every input a later decision reads defined where the verb takes it, and does each stated bound match what the guard in internal/loomcli/decision.go actually checks?"
---

Round 1 ran all four lenses with MEDIUM or worse findings, so no lens is excluded.
