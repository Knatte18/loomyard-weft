# Card 9 — arbiter-orch-stencils

**Edit:**
- `contracts/stencils/loom/loom-template-arbiter.md`
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`

**Intent:** The arbiter's fixed review rule follows the escalation's cause (D7), and the orch's escalation guidance follows the hard cap (D9).

`loom-template-arbiter.md`, section "When you hand on": the bullet "At a review escalation the rule is fixed: accept unless a finding makes the design wrong, and hand on a blocker that remains after round 3" becomes a rule per cause, the cause read from the review escalation brief.
What a ledger entry's `design-wrong` and `mechanical` tags mean is the judge stencil's, which the brief names, and the arbiter stencil restates neither (`PATTERN-producer-pointer-rule`):

- `hard-cap`: read the open design-wrong findings against the code; accept when a finding is wrong or the fix in place answers it, otherwise hand on.
- `circling`: accept when the returned finding is wrong; continue when the fix is converging and the round is below the hard cap; otherwise hand on.
- `budget`: settle from the review, ledger and verdict files as for `circling`, or hand on.

The hand-on itself goes to the orch as today, and the "What you may change" list keeps both circling verbs, since `circling continue` refuses at the hard cap by itself and a refusal makes the arbiter hand on.

`orch-template-role.md`, section "Escalations from a child":

- The bullet "A review loop that ran out of rounds: `lyx loom circling continue <slug>` to grant more, …" says `lyx loom circling continue <slug>` grants one more round only below the perch's hard cap.
  At or past it, or at a `hard-cap` escalation, the way forward is `lyx loom circling accept <slug>`, or raising the hard cap by the command the refusal names for the layer its value comes from.
  The "Either only records the decision" lines stay.
- The board section's small-task bullet advises `run_settings` `{"loom": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}`.
  It keeps "never 1, since a second round reviews the first round's fixes" and adds that a hard cap of 3 lets the soft-cap rule buy one reviewed round for a design-wrong fix and no more.

`wordingClaims`:

- The `loom-template-arbiter.md` row drops "after round 3" from its must-claims and gains must-claims `` `hard-cap` ``, `` `circling` ``, `` `budget` ``, "below the hard cap" and "judge stencil", with a why naming D7.
  It gains a must-not-claim "after round 3", so the old fixed rule cannot return.
- The `orch-template-role.md` row gains must-claims `"review_hard_max_bounces": 3`, "only below the perch's hard cap", "at a `hard-cap` escalation" and "raising the hard cap", the last two scoped to "## Escalations from a child", and a must-not-claim "to grant more", each with a why naming D9.
  Its existing "after round 3" claim on the orch's own hand-on bullet in "Escalations from a child" stays, since D9 leaves that bullet unchanged.

**ImpactSummary:** Wording of two shipped stencils and their claims rows; no marker changes, so no Go fill site changes.

**Commit:** 9: arbiter rule per escalation cause; orch guidance and small-task advice follow the hard cap
