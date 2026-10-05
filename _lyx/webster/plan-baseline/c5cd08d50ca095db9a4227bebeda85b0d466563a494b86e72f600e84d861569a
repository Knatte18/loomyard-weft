# Card 6 — Escalation brief and parent-notice stencils

**Create:**
- `contracts/stencils/bouncer/bouncer-template-escalation.md`
- `contracts/stencils/bouncer/bouncer-template-parent-notice.md`

**Edit:**
- `contracts/stencils/stencils.go`
- `contracts/stencils/registry_test.go`
- `contracts/stencils/claims_test.go`

**Intent:**
D5, D7, D9, D10: the wording of the escalation brief and the parent notice lives in stencils read from the told stencils directory (`PATTERN-stencil-ownership`), never in Go.
Both are conditional-free, with top-level markers only, like the judge stencil.

`bouncer-template-escalation` is the brief a one-shot fork of the parent session reads.
Markers: `segment`, `round`, `cause`, `review_path`, `ledger_path`, `verdict_path`, `decision_record_path`, `worktree`, `slug`.
It states the cause (`circling`: the judge found no progress on recurring gating findings; `budget`: the review budget is spent without convergence), lists the materials to read, and instructs the fork to:
1. record each design call it makes with `lyx loom decision add {{.slug}} --by parent --title … --decision … --rationale …`;
2. record `lyx loom circling accept {{.slug}}` (end the segment unconverged on this round) or `lyx loom circling continue {{.slug}}` (run one more round);
3. resume the run with `lyx loom start` in `{{.worktree}}`.
When it cannot settle the question it records nothing and reports back to its parent session, which asks the operator; the run stays `awaiting`.
It names that `accept` passes the segment with its open findings, recorded as unconverged.

`bouncer-template-parent-notice` is one line with markers `slug` and `brief_path` only, asking the parent to review the brief with a one-shot fork; it interpolates no judge, review or ledger text.

Register both in `stencils.go` (embedded vars plus registry rows beside the other two Bouncer stencils), extend `registry_test.go`'s Bouncer name list, and add `claims_test.go` rows: the brief names `lyx loom decision add`, both `circling` verbs and `lyx loom start`; the notice names `{{.brief_path}}` and carries none of `{{.review_path}}`, `{{.ledger_path}}`, `{{.verdict_path}}`.

**ImpactSummary:** Two new registered stencils seeded with the others; nothing renders them until Card 10.

**Commit:** 6: Escalation brief and parent-notice stencils
