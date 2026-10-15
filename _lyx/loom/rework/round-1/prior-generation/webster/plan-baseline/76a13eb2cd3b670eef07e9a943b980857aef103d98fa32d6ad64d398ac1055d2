# Card 1 — critical-judge

**Edit:**
- `contracts/stencils/twill/twill-directive.md`
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `internal/twilldirective#filledValues`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/twilldirective#TestDirective_MissingValueNamesTheMarker`
- `internal/twilldirective/doc.go`
- `internal/loomengine#planLensDirective`
- `internal/loomengine#discussionLensDirective`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestDiscussionTable`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `docs/overview.md`

**Delete:**
- `internal/loomengine#planTwillRejectionRule`

**Intent:**
Replace the twill directive's judge step with the critical fixer (D6), make the fork boilerplate host-neutral (D5), and replace the re-prompt sentence with the host-neutral re-prompt rule (D3), for both hosts.
The two relaunch paragraphs stay in the directive in this card; card 2 moves them.

Stencil `twill-directive.md`:

- Step 4 becomes the judge below, `{{.rejection_rule}}` retires, and two new required markers arrive, `{{.governing_input}}` and `{{.dispute_evidence}}`:
  "4. Judge every finding: merge duplicates across lenses, then weigh each finding twice: its premise, against the code and {{.governing_input}}, and the change it proposes.
  Give it exactly one of three dispositions, decided in this order; severity decides how a finding is reported, not whether it is weighed."
  Then three sub-bullets.
  **Dispute**, on one of two grounds, `premise false` (the premise is false against what you read) or `every fix worse` (the premise holds, but every change that addresses it makes the reviewed files worse); one sound change means the finding is fixed, not disputed; the evidence is cited text opened in this session, {{.dispute_evidence}}, and what it shows against the finding; severity, size, cost, taste, and disagreement with the rubric or `PATTERN` never justify a dispute; a finding you cannot refute from what you read is not disputed; keeping a waiver, a deferral or an unplanned test against a finding is a dispute and needs the same evidence.
  **Fix with your own change**: the premise holds, the change the finding proposes is wrong or harmful, and another change resolves the premise; make that change, and name the proposed change you did not make and why, with the same kinds of evidence.
  **Fix as proposed**: otherwise; a finding that proposes no concrete change leaves the change to you.
  Follow the benched `critical-judge.md` wording, without Burler's write-surface deferral.
- Step 5 becomes: "Record the lens review at `{{.disposition_place}}`: one line per finding, with its lens, its severity and its disposition: `disputed` with its ground and the evidence, `fixed differently` with the proposed change you did not make, why, and what changed instead, or `fixed as proposed` with what changed." followed by the unchanged "Name a fork that returned no report."
- The sentence "A gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again." is removed, and in its place, after "One fork pass per writer spawn.", the re-prompt rule: "A writer re-prompted by a gate or a parent-review reject fixes the reviewed files itself: if it has not yet made its lens spawn call (step 2), it runs this procedure once; otherwise it spawns no forks."
- The boilerplate's first line becomes "- {{.reviewed_files}} were written and passed the self-check before you were spawned."; the rest of the boilerplate stays.
- The banner comment's marker list follows the new set.
  No new text may carry a `hostNeutralClaims` phrase, matched in any case (so never "step 5" or "step 6").

`Values` loses `RejectionRule` and gains `GoverningInput` (the phrase naming the input a finding's premise is weighed against, rendered verbatim) and `DisputeEvidence` (the phrase listing the evidence kinds a dispute cites, rendered verbatim after "opened in this session,").
`Directive` maps them to `governing_input` and `dispute_evidence` and drops `rejection_rule`; every marker stays required.

Hosts:

- `planLensDirective` sets `GoverningInput` to "the decision record" and `DisputeEvidence` to "a `path:line` or glyph, a `PATTERN` entry, a test or caller the change would break, or the `lyx loom validate-plan` output".
  `planTwillRejectionRule` is deleted; its only users are `planLensDirective` and `TestPlanTable`, both on this card.
- `discussionLensDirective` drops its rejection rule and sets `GoverningInput` to "the board entry" and `DisputeEvidence` to "the board entry, the discussion spec, the rubric, a `PATTERN` entry, the `lyx loom validate-discussion` output, or a coarse code read within your Step 2 bound, with no `path:line` required".
- The severity rules stay per host.

Tests:

- `filledValues` drops `RejectionRule` and sets `GoverningInput: "GOVERNING-INPUT"` and `DisputeEvidence: "DISPUTE-EVIDENCE"`.
  `TestDirective_RendersEveryValueVerbatim` swaps the `REJECTION-RULE-SENTENCE` want for "against the code and GOVERNING-INPUT" and "opened in this session, DISPUTE-EVIDENCE, and what it shows", and the "wrote `/abs/one.md` and `/abs/two.md` and passed" want for "`/abs/one.md` and `/abs/two.md` were written and passed the self-check before you were spawned".
  `TestDirective_MissingValueNamesTheMarker` drops its `RejectionRule` row and adds a `GoverningInput` row naming `governing_input` and a `DisputeEvidence` row naming `dispute_evidence`.
- `TestStencils_TopLevelMarkers`'s `twill-directive` row drops `rejection_rule` and adds `dispute_evidence` and `governing_input`.
- `wordingClaims`' `twill-directive.md` row: the `{{.rejection_rule}}` claim becomes `{{.governing_input}}` and `{{.dispute_evidence}}` claims; "passed its self-check before spawning you" becomes "were written and passed the self-check before you were spawned"; new must claims "exactly one of three dispositions", "`premise false`", "`every fix worse`", "One sound change means the finding is fixed, not disputed", "never justify a dispute", "**Fix with your own change**", "**Fix as proposed**", "`fixed differently`" and "if it has not yet made its lens spawn call"; a new mustNot claim "a later turn of the same session", why: the re-prompt rule replaces that sentence (D3).
- The host tests pin what each host supplies and where it lands; the stencil-only wording is pinned once, by `wordingClaims`.
  `TestPlanTable`'s "lens review" subtest drops the `planTwillRejectionRule` want and adds "against the code and the decision record", the plan `DisputeEvidence` phrase and "`" + overview + "` and every card file its Card Index names were written and passed the self-check before you were spawned.", and asserts the prompt lacks "a later turn of the same session".
  A lost host value or a surviving old sentence each fails one of these.
- `TestDiscussionTable`'s "lens review" subtest swaps the "The writer wrote …" want for "`<record>` and `<support log>` were written and passed the self-check before you were spawned." and the "Fix the record, or reject the finding with a one-line reason." want for "against the code and the board entry" and the discussion `DisputeEvidence` phrase (which carries "with no `path:line` required"), and asserts the prompt lacks "a later turn of the same session".
  Its existing "runs `lyx loom validate-discussion` per `Step 6` until it passes" want keeps pinning the relaunch paragraphs at this host.
  Its two-advisor `lensKeys` loop additionally asserts the lead prompt carries "against the code and the board entry" exactly when `wantDirective`; the loop's existing check that no advisor prompt carries "## Lens review by forks" already keeps the judge out of the advisors.

Docs, in this commit: the `twilldirective` package doc's host-value list names the governing input and the dispute evidence kinds instead of the rejection rule, and says the writer judges each finding as a critical fixer (dispute on a cited ground, fix with its own change, or fix as proposed).
`docs/overview.md`'s Plan-Write lens line ("rejects a `BLOCKING` or `MEDIUM` finding only with a cited code read") and its Discussion-Write lens line ("takes the forks' findings in and finalises") say the lead judges each finding as a critical fixer and records its disposition.

**ImpactSummary:** Retires the rejection_rule marker and Values field for two new required host values; both twill hosts, the directive stencil, its claims and markers pins, and the two loomengine host tests move together, with no caller outside these packages.
