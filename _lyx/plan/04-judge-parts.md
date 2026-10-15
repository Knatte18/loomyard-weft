# Card 4 — judge-parts

**Create:**
- `contracts/stencils/twill/twill-judge-critical.md`
- `contracts/stencils/twill/twill-judge-plain.md`
- `contracts/stencils#TwillJudgeCritical`
- `contracts/stencils#TwillJudgePlain`
- `internal/twilldirective#CriticalJudge`
- `internal/twilldirective#criticalJudgeStencil`
- `internal/twilldirective#plainJudgeStencil`
- `internal/twilldirective#filledCriticalJudge`

**Edit:**
- `contracts/stencils/twill/twill-directive.md`
- `contracts/stencils#entries`
- `contracts/stencils#TestStencils_TopLevelMarkers`
- `contracts/stencils#wordingClaims`
- `internal/twilldirective#Values`
- `internal/twilldirective#Directive`
- `internal/twilldirective#filledValues`
- `internal/twilldirective#TestDirective_RendersEveryValueVerbatim`
- `internal/twilldirective#TestDirective_MissingValueNamesTheMarker`
- `internal/twilldirective#TestDirective_MissingStencilNamesIt`
- `internal/twilldirective/doc.go`
- `internal/loomengine#planLensDirective`
- `internal/loomengine#discussionLensDirective`
- `internal/loomengine#TestDiscussionTable`
- `docs/overview.md`

**Uses:**
- `internal/twilldirective#fillPart`
- `internal/twilldirective#mergedValues`
- `internal/loomengine#TestPlanTable`
- `contracts/stencils#hostNeutralClaims`

**Intent:**
Restrict the critical-fixer judge to Plan-Write, as the decision record's last entry ("Critical fixer only at Plan-Write") decides: Discussion-Write gets today's judge steps from `main` back, byte for byte, while both hosts keep the host-neutral re-prompt rule (D3) and the D5 fork boilerplate.
The judge becomes a host-chosen part, the way the writer-fork and relaunch parts already are.

Stencils (both new parts host-neutral, no `{{if}}`, each opening with a banner comment in the `twill-relaunch.md` style naming its required markers):

- `twill-judge-critical.md` holds today's steps 4 and 5 of `twill-directive.md` moved verbatim: the critical-fixer step 4 and the step 5 that records each disposition, ending with "Name a fork that returned no report."
  Its markers are `{{.governing_input}}`, `{{.dispute_evidence}}` and `{{.disposition_place}}`.
- `twill-judge-plain.md` holds `main`'s steps 4 and 5 verbatim (`git show main:contracts/stencils/twill/twill-directive.md`):
  "4. Judge every finding with the same skepticism, and merge duplicates across lenses." / "   {{.rejection_rule}}" / "5. Record the lens review at `{{.disposition_place}}`: one line per finding, with its lens and whether it was fixed or rejected and why." / "   Name a fork that returned no report."
  Its markers are `{{.rejection_rule}}` and `{{.disposition_place}}`.
- `twill-directive.md` replaces those two steps with one required marker `{{.judge_steps}}` on its own line, between step 3 and "6. Run `{{.final_self_check_command}}` again …", so a filled part without trailing newlines leaves the step list's bytes as they were.
  Its markers become `final_self_check_command`, `judge_steps`, `lens_list`, `relaunch_rules`, `reviewed_files`, `rubric_path`, `severity_rule`, `spawn_point` and `writer_fork`; its banner comment lists them and says `judge_steps` is filled by `Directive` with one of the two judge parts.
  The marker is required, not optional: every host gets exactly one judge part, so an unfilled marker is a wiring fault the fill must refuse.

`contracts/stencils`: embed the two files as `TwillJudgeCritical` and `TwillJudgePlain`, with doc comments in the `TwillRelaunch` style, and register `twill-judge-critical` and `twill-judge-plain` in `entries` after `twill-relaunch`, so `TestRegistry_MatchesOnDiskTree` and `stencilkit.Seed` see them.

`internal/twilldirective`:

- `CriticalJudge` holds the two host values only the critical part needs, each rendered verbatim: `GoverningInput` and `DisputeEvidence`, moved off `Values` with their doc comments.
- `Values` loses `GoverningInput` and `DisputeEvidence` and gains `RejectionRule` (the sentence that tells the writer how to fix or reject a finding, rendered verbatim on its own line under step 4, as on `main`) and `CriticalJudge *CriticalJudge`: nil asks for the plain judge, which needs `RejectionRule`; set, it asks for the critical judge and `RejectionRule` is not read.
- `Directive` adds `rejection_rule` to the shared value map, then fills exactly one judge part through `fillPart`: with `CriticalJudge` set, `twill-judge-critical` over the map merged with `governing_input` and `dispute_evidence`; otherwise `twill-judge-plain` over the map.
  The filled part becomes the `judge_steps` value, with nothing appended; the writer-fork and relaunch choice is unchanged.
  The error branches are `fillPart`'s existing ones: an empty `RejectionRule` with no critical judge, or an empty `GoverningInput` or `DisputeEvidence` with one, fails the part's fill with an error naming the marker and the part's stencil; a missing judge stencil fails naming it.
  Each is reached by a row below after every other value is filled.
- The package doc's host-value list names the rejection rule, and, for a host that asks for the critical judge through `Values.CriticalJudge`, the governing input and the dispute evidence kinds.
  It says exactly one judge part is rendered, each from its own stencil: the critical fixer (dispute on a cited ground, fix with its own change, or fix as proposed) for a host that asks for it, otherwise the judge that fixes or rejects each finding by the host's rejection rule.
  The sentence "The writer judges each finding as a critical fixer: …" goes.

Hosts:

- `planLensDirective` moves its `GoverningInput` and `DisputeEvidence` values, unchanged, into `CriticalJudge: &twilldirective.CriticalJudge{…}` and sets no `RejectionRule`.
- `discussionLensDirective` drops `GoverningInput` and `DisputeEvidence` and sets `RejectionRule: "Fix the record, or reject the finding with a one-line reason."`, its value on `main`, with no `CriticalJudge`.

Tests (no new top-level test; every case is a row or assertion of an existing test):

- `filledCriticalJudge` returns a `CriticalJudge` with `GoverningInput: "GOVERNING-INPUT"` and `DisputeEvidence: "DISPUTE-EVIDENCE"`.
  `filledValues` drops those two fields and sets `RejectionRule: "REJECTION-RULE-SENTENCE"`.
- `TestDirective_RendersEveryValueVerbatim`: the shared wants drop "against the code and GOVERNING-INPUT" and "opened in this session, DISPUTE-EVIDENCE, and what it shows".
  They gain "forks are outstanding is allowed.\n4. Judge every finding" and "returned no report.\n6. Run `lyx check --final` again", which both judge parts satisfy and which pin the part's placement with no blank line on either side at both hosts.
  The "without a writer fork" row, which mirrors Discussion-Write, wants "   REJECTION-RULE-SENTENCE\n5. Record the lens review at `the PLACE section`".
  The "with a writer fork" row, which mirrors Plan-Write, also sets `filledCriticalJudge()`, wants the two dropped strings and lacks "REJECTION-RULE-SENTENCE", so a `Directive` that renders the plain part beside the critical one fails.
- `TestDirective_MissingValueNamesTheMarker`: the `GoverningInput` and `DisputeEvidence` rows set `filledCriticalJudge()` and blank that field, still naming `governing_input` and `dispute_evidence`; a new `RejectionRule` row without a critical judge blanks it and names `rejection_rule`.
- `TestDirective_MissingStencilNamesIt` gains four rows: `twill-judge-critical` removed with a critical judge and `twill-judge-plain` removed without one each fail naming the stencil; `twill-judge-plain` removed with a critical judge and `twill-judge-critical` removed without one each render cleanly, so a `Directive` that reads both judge parts fails.
- `TestStencils_TopLevelMarkers`: the `twill-directive` row gets the new set; new rows `twill-judge-critical` (`disposition_place`, `dispute_evidence`, `governing_input`) and `twill-judge-plain` (`disposition_place`, `rejection_rule`).
- `wordingClaims`: the critical-fixer claims ("exactly one of three dispositions", "`premise false`", "`every fix worse`", "One sound change means the finding is fixed, not disputed", "never justify a dispute", "**Fix with your own change**", "**Fix as proposed**", "`fixed differently`"), "no report", `{{.governing_input}}`, `{{.dispute_evidence}}` and `{{.disposition_place}}` move from the `twill-directive.md` row to a new `twill-judge-critical.md` row joined with `hostNeutralClaims()`.
  A new `twill-judge-plain.md` row joined with `hostNeutralClaims()` claims "with the same skepticism", "whether it was fixed or rejected and why", "no report", `{{.rejection_rule}}` and `{{.disposition_place}}`.
  The directive row adds `{{.judge_steps}}` and keeps its re-prompt claim ("if it has not yet made its lens spawn call"), its mustNot "a later turn of the same session" and its boilerplate claim ("were written and passed the self-check before you were spawned").
- `TestDiscussionTable`: the two-advisor `lensKeys` loop drops its "against the code and the board entry" assertion, back to `main`'s loop.
  The "lens review" subtest swaps its "against the code and the board entry" and Discussion `DisputeEvidence` wants for `main`'s "Fix the record, or reject the finding with a one-line reason.", and keeps its D5 boilerplate want and its mustNot loop over "a later turn of the same session" and "fork0".
- `TestPlanTable` needs no edit: its "against the code and the decision record" and Plan `DisputeEvidence` wants keep pinning the critical part at Plan-Write.

Docs, in this commit: `docs/overview.md`'s Discussion-Write lens line goes back to `main`'s "… forks itself once per lens of that fan, takes the forks' findings in and finalises."; the Plan-Write lens line keeps the critical fixer.
`internal/loomengine/plan.go`'s file comment and `internal/loomengine/template.yaml`'s `plan_lenses` comment describe Plan-Write only and stay.

**ImpactSummary:** Moves the twill judge steps into two registered part stencils chosen per host through a new `Values.CriticalJudge`; Plan-Write's rendered prompt is unchanged, Discussion-Write's judge steps return to `main`'s bytes, and only the twilldirective, loomengine and contracts/stencils tests and the two docs move.
