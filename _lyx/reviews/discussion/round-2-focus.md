---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md`, D7 (advisor start failure and advisor death notices to the chair), D8 (advisors start in table order, the chair last) and D9: settle whether every notice the engine raises about an advisor has a defined recipient and delivery path at the moment it is raised, including a notice raised before the chair has a strand or handle on a fresh start."
  - "In D7, the Go-side notice sent on the chair's handle: settle whether the delivery path matches how `Run.Send` in `internal/shuttleengine/run.go` behaves (it waits for an idle session and fails with `ErrSessionBusy` past the send-ready window) against D6's rationale that the chair keeps drafting instead of ending its turn, and whether the Decision names the goroutine or loop that retries and what bounds each retry."
  - "In D5 (stop on the chair's terminal result), D7 (advisor reaches done) and D2's `Seat` fields: settle what an advisor's strand and run directory look like after the advisor writes its output file, given that `finalize` in `internal/shuttleengine` removes the strand and run directory on Done unless `Spec.KeepPane` is true, and whether the chair can still reach that advisor afterwards as the Goal's ask-and-answer channel requires."
  - "In D3 (every seat renders the parent directive), D6's bound and the advisor block: settle what an advisor may send its parent through the `SendMessage` route that `contracts/stencils/parent/parent-directive-parent.md` gives it, what bounds that edge, and whether the Goal's claim that advisor traffic travels over a channel the parent never sees still holds."
  - "In D5, the meaning of an advisor 'still live' when the chair settles: settle whether an advisor that timed out or died before the chair, which keeps its strand in reed, is stopped, given one `Timeout` governs every seat and advisors start before the chair."
---

Focus targets the BLOCKING and gating-class MEDIUM design questions and the MEDIUM consistency question of the advisor lifecycle.
The LOW consistency items (provider-word claims, the "Added after Discussion" section, argued alternatives in rationales) stay in scope as ordinary review items.
