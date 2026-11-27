```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review's proposals that the fixer carried out (F1–F5).

**1. MEDIUM: F3 texts state a reservation the code does not make**
- **Changed text:**
  - `_lym/landing/summary.md`, the design-decision bullet: "the review gets time on top of the writer's budget".
  - `internal/landingshed/describe.go`, the `DescribeTable` doc: "so the lead's deadline leaves the review its time after landing.describe_timeout_min".
  - `describereview.go`, the `describeReviewSeat` doc: "the review then has time of its own".
- **What is wrong:** the proposal only raises both limits to the sum. Nothing holds the writer to `describe_timeout_min`: `table.Seats[0].Timeout = reviewer.Timeout` gives the writer all of the 42 or 45 minutes.
- **Concrete case:** with the 30 and 15 defaults, a writer that first ends a passing turn at minute 44 leaves the review 1 minute. `classifyDeadlineExpiry` then finalizes Done through the entry's `Final` pass, and the text lands unreviewed. This is the review's F3 scenario, moved from minute 30 to minute 45.
- **Second case:** the writer ends at minute 30, the review takes 14 minutes, and the findings arrive at minute 44. The writer has 1 minute to fix them before the same deadline.
- **What holds:** the review gets the extra time only if the writer finishes within `describe_timeout_min`, and the fix turn comes out of the same sum.
- **Smallest remedy:** word those three texts conditionally. A real reservation would need a separate writer cap, which the seat's single `Timeout` does not express.
- **Already accurate:** `template.yaml`, `config.go` and `overview.md` state only the limits.

**F1, F2, F4, F5:** I found no case where the proposal is wrong or breaks something.
- **F2:** the way forward also covers `completeSummary` failures ("fix what the cause names"), which the review's own wording did not.
- **F4:** each of the three forwards, if dropped, would fail one of the new assertions.
- **F5:** the added rule does not contradict the beam stencil's other content rules.

I made no further tool calls.
