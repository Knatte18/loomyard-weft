---
round: 3
focus:
  - "decision-record.md D5, revive versus a hub pause: settle whether the stated bound on a pause entry added after revive's coverage read and before its line lands holds when the covered sessions are idle-stalled (the pause polls until every covered session reads idle, then records and interrupts only sessions reading a running turn, per internal/pauseengine/doc.go), and whether the mechanism chosen closes the window or only narrows it."
  - "decision-record.md D5 and Scope In: settle whether every shuttleengine behavior D5 relies on for revive's send (a state-and-cause precondition, refusing with no idle wait) exists today in Runner.Send / sendVerified / awaitIdleSession (internal/shuttleengine), and if not, whether Scope In and the docs-in-the-same-commit constraint name that change and its bound."
  - "decision-record.md D6, held calls: settle whether every cancellation claim and Acceptance criterion about a call cancelled after release is achievable when the caller's context was derived with context.WithTimeout and the hold outlasted that deadline, so the request context is already done."
  - "decision-record.md D8, Bound: settle whether the wording about what the new orch watcher image does with a live hub pause entry from its first tick matches internal/orchengine/pause.go, where settledWithoutInterrupt treats an entry whose GraceEnd precedes the first tick as settled."
---

Each entry restates its sites so the reviewer needs no earlier round.
