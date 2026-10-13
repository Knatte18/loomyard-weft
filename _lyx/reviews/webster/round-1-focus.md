---
round: 1
focus:
  - "Bouncer hard-cap logic (card 3): check that the cap is compared against the segment's current-generation round number and never Shed's bounce count, so a goto reset cannot pass it. Also check that continues past the soft cap are exempt, and that HardCap 0 leaves behavior unchanged."
  - "Soft-cap rule (card 5): check the design-wrong test (open, BLOCKING or MEDIUM, not tagged mechanical) against the Shared Decisions. Check that auto-accept's replay and blocking paths settle the same way, and that unreadable inputs escalate instead of silently accepting."
  - "Config load refusals (cards 6 and 7): check that a cap below 1 or below the soft cap is refused in every layer with the right per-layer route. Check that the resolved cap reaches each Bouncer row through Env.RowHardMaxBounces."
  - "circling continue refusal (card 8): check that it refuses at and past the perch's hard cap and agrees with the Bouncer's own cap check (card 3), with no off-by-one between the two."
  - "Escalation cause plumbing (cards 2 and 9): check that hard-cap round-trips through the escalation record and the circling decision file. Check that the two stencil fragments are chosen by hard cap in Go, with Go interpolating values only, and that escalations at the hard cap offer only accept."
  - "Test economy: check each new test for duplication of an existing test's coverage, and each added behavior for missing coverage."
---

Seed focus drawn from the plan's Shared Decisions and card index; the review range is the branch's own commits over its merge base with the parent, since no rework round exists.
