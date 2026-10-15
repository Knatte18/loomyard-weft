# Card 8 — Exempt stencil prose from the coverage rule and have review guard it

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `pattern/PATTERN-test-economy.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `contracts/stencils/loom/loom-rubric-darn-review.md`
- `contracts/stencils/loom/loom-rubric-merge-review.md`

**Intent:**
Change the rules that ask for a test per stencil change (D3), and make review guard prompt wording (D4), one sentence per line in every edited file:

- `loom-plan-spec.md`'s "What a card is" rule 3 adds one line: a stencil's static prose is not behavior a test covers; a stencil card carries a test only when it changes a marker, a value Go fills, reads or names, a fragment Go chooses, or a rule a scan across stencils checks.
- `loom-template-plan.md`'s item 3 ("Carries the coverage for the behavior it introduces") carries the same exemption in one line, saying no more of the spec than the item already does (`PATTERN-producer-pointer-rule`).
- `pattern/PATTERN-test-economy.md` gains one bullet: no test pins a stencil's static prose; a stencil test may pin only a marker set, a Fill round-trip or a Go-filled marker's position, a value Go fills, reads or names, a fragment Go chooses, or a rule across a stencil family, and review guards the wording.
  The `PATTERN.md` one-liner is unchanged.
- `loom-rubric-plan-review.md`'s **Test economy** bullet adds: a card that plans an assertion on a stencil's static prose is a finding.
- The **Test economy** lists of `loom-rubric-webster-review.md` and `loom-rubric-darn-review.md` gain one item, a new assertion on a stencil's static prose, and their "a behavior the diff adds with no coverage" item exempts a stencil's static prose; `loom-rubric-merge-review.md`'s "a behavior of the interaction with no coverage" item exempts it too.
- `loom-rubric-webster-review.md` and `loom-rubric-darn-review.md` each gain one `## Also flag` bullet, **Stencil wording**: a diff that changes a stencil is read against the measuring stick (the plan card's `Intent` in the Webster-Review rubric, the board entry as the decision record's added entries amend it in the Darn-Review rubric), and a dropped, weakened or contradicted rule the measuring stick did not ask to change is a finding.

No rubric gains a marker, so `TestLoomRubrics`' allowlist holds, and every spec citation keeps its `{{.specs_dir}}` prefix, so the citation scan holds.

The guard this downgrades, from a mechanical substring test to a judged review, now lets through a prose regression the reviewer misses, a stencil change landed outside loom and darn (an orchestrator's own pushed fix, an operator commit on `main`), a loom run whose Webster-Review budget is 0, a darn run whose `darn.yaml` `review_max_bounces` is 0, a loom run with `webster_auto_accept` true at a spent budget, and a round at or past the hard cap.
What bounds it: in a loom landing whose Webster-Review budget is above 0 the reviewer reads every stencil change against the plan card's `Intent` and a finding is fixed or escalated, unless the perch's auto-accept or hard cap accepts the round; in a darn landing whose `review_max_bounces` is above 0 Darn-Review reads it against the board entry likewise, unless the hard cap accepts the round.
