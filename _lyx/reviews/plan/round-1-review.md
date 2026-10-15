---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-darn-own-entry-rule.md
    summary: Card 4's darn-review bullet treats a step in the decision record as evidence that the writer made an own-entry change, yet the same card's darn route has the writer leave out a step the record asks for, so a writer who follows the route would be flagged
    fix: In Card 4's Intent, in the paragraph on the loom-rubric-darn-review.md bullet, replace the evidence clause "a claim in the change description or a step in the decision record that changes this task's own entry's body or scope" with "a claim in the change description that the run changed this task's own entry's body or scope", and add one sentence saying that a decision-record step asking for such a change is evidence only of the request, which the next sentence of that paragraph (left out through the own-entry route, no scope finding once recorded, MEDIUM decision before then) governs.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/plan/02-discussion-own-entry-rule.md
    summary: Card 2 places the new Discussion-stencil sentence in Step 3's paragraph closing Design the full scope now, but that bold sentence opens the last three lines of Step 3's paragraph rather than closing it
    fix: In Card 2's Intent, replace "(Step 3's paragraph closing \"Design the full scope now.\" is the natural home)" with "(Step 3's closing paragraph, the one holding \"**Design the full scope now.**\" and its no-MVP and YAGNI sentences, is the natural home)".
---

### [MEDIUM] Darn-review evidence clause conflates a requested own-entry step with a performed one

**Location:** `_lyx/plan/04-darn-own-entry-rule.md`, the paragraph on `loom-rubric-darn-review.md`

**Issue:** The lens fix made the new darn-review bullet name its evidence as "a claim in the change description or a step in the decision record that changes this task's own entry's body or scope".
In a darn run the decision record is the writer's pre-coding claim, and it can hold such a step: D4 and Card 4's own darn route say the writer acts "when the board entry or the decision record asks for such a step" by leaving the step out.
A record step therefore shows that the step was requested, not that it was done.
Read as written, the bullet makes the reviewer file a finding against a writer who followed D4's route.
That contradicts the card's next sentence, which says such an omission is no scope finding once recorded and a MEDIUM `decision` finding before then.
D5 asks the rubric to flag a darn step that *changes* the own entry, and the change description is the only artifact that claims what the run did.
No targets or verify command change, so no `validate-plan --overlay` run applies.

### [NIT] Card 2 misdescribes where the Discussion-stencil sentence goes

**Location:** `_lyx/plan/02-discussion-own-entry-rule.md`

**Issue:** `contracts/stencils/loom/loom-template-discussion.md:73` holds `**Design the full scope now.**`, followed by the no-MVP sentence and the YAGNI sentence before `## Step 4`.
The card calls it the sentence that closes the paragraph, so an implementer looking for that sentence at the paragraph's end will not find it there.
The intended home is clear enough, so this is a wording slip only.

## Focus verdicts

- Attack surface, Cards 3 and 4: not a defect.
  Card 3 names what the widened route skips, which is a decided own-entry step, and states the bound, "admits a drop of an own-entry step only, and no other decided step is dropped through it".
  Its Fidelity amendment, which downgrades a guard, keeps "any other uncarried Decision stays a finding as before".
  Card 4 states the same bound for the darn route, and its rubric text keeps the pre-record omission as a MEDIUM `decision` finding.
- Fidelity to D3, D4 and D5: every sentence is carried.
  Card 3 has the Plan writer wait for the `Added after Discussion` entry and stop and report with no parent.
  Card 4 has the darn writer leave the step out from the start, name it as an unrecorded parent correction until the entry exists, and stop only with no parent.
  Both match D4's rationale for the asymmetry.
  No card adds scope the record does not license.
  Each card's bound list matches the Constraints' "body or scope" rule and the Scope's Out list.
- Card 3's rubric amendment and the symmetry note: consistent.
  The Plan stencil plans without the step only once the record carries the entry, and the amended Fidelity bullet accepts the missing step under that same condition, so nothing the stencil tells the writer to omit is flagged.
- Card 1: the two lines that imply a cut, `CLAUDE.md:47` and `docs/overview.md:95-96`, are the only matches for that wording under the repo's `.md` files, and the card rewrites both.
  `docs/overview.md:102` says only where unbuilt work lives.
  Three docs link to `overview.md#documentation-lifecycle`, and keeping the heading byte-identical keeps the slug `internal/lyxcwd/docslink_test.go` resolves.
- Verify coverage: covered.
  The repository has one root `go.mod` for these paths, so `go test ./...` runs `contracts/stencils` (including `TestStencils_NoBareCrossRepoCitations`, `TestStencils_NoUnboundedAskOperator` and `TestStencils_NoEditRuleRestatement`) and `internal/lyxcwd`'s untagged docs link test.
  The section follows the template's order.
- Test economy: no card plans a test or a static-prose assertion, and since no top-level test is added, `cmd/lyx/testdata/test-budget.yaml` needs no listing.
  No other test or golden file holds the anchor sentences the cards change; a grep for each found only the stencil itself and `docs/research/` notes.
