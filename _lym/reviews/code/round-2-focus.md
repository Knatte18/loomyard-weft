---
round: 2
focus:
  - "internal/dobbyengine/runnotice_test.go, TestRunNotice_Conditions: is there a row with a dead driver, a pause record whose Intervals hold a paused interval, and no expected notice, such that dropping pause.Intervals from the AwakeSince call in noticeCondition (internal/dobbyengine/runnotice.go) makes it fail?"
  - "internal/dobbyengine/doc.go and internal/battenshed/doc.go: do the driver-dead notice paragraphs name the loom bootstrap lock as a live-driver signal and the minute of awake time the notice waits from the running episode's since, including on a takeover that dates since from first sight?"
  - "_lym/landing/summary.md, Design decisions: does the claim about when the driver-start grace applies match runNotice, which restarts memory.Since only when dobby sees a non-running child state?"
---

Round 1's gating finding concerned test coverage of the grace's pause handling.
Round 2 reviews the tree fresh, with these sites as the first places to look.
