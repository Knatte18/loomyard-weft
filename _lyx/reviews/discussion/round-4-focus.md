---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md`, the Decision on the bounce budget (D4): does it state one budget-reached test that matches Shed's own comparison (`episodeStuckCount(st.History, ...) >= effectiveMaxBounces` in `internal/shedengine/run.go`, counted before the current Stuck is appended, with the Bouncer's seed Stuck already counted per `internal/shedadapters/bouncer.go`), and is that boundary consistent with the Goal and the acceptance criteria about the Nth judged round of N?"
  - "In the same D4, the claim that a circling `continue` cannot reach Shed's `bounce-budget-exhausted` block: does it hold given that the `Bounces` seam is optional in `internal/shedadapters/bouncer.go`, and does the Decision state what is wired and what residual halt remains when the seam is unwired or fails?"
  - "In the Goal and the Decision that bounds additions to the decision record (D8): which reader actually checks an entry marked as added after Discussion? `contracts/stencils/loom/loom-rubric-plan-review.md` is the Plan-Review rubric that reads the decision record; does the stated bound claim only what the decided work provides, with no reviewer assumed that no Decision or Scope item introduces?"
---

Each entry names one boundary or claim to settle at its root, so the wording does not need another patch.
