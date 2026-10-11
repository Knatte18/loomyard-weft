---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/03-carry-over-fixed-or-disputed.md
    summary: Card 3 leaves two claim whys in contracts/stencils/claims_test.go that still call a carry-over entry an unreviewed fix, which D7 makes false
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-fixer-dispositions.md
    summary: Card 1 has each new claim's why cite D1 to D3 of this task's decision record, a file the code repository does not track, so the provenance dangles once the task lands
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/00-overview.md
    summary: The one-rule-many-sites Shared Decision has the fix orchestrator's Validate step state the disposition text, contradicting Card 1's Intent and PATTERN-producer-pointer-rule
---

### [LOW] Card 3 misses the claim whys that still describe a carry-over entry as an unreviewed fix

**Location:** `_lyx/plan/03-carry-over-fixed-or-disputed.md`

**Issue:**
D7 retires the text that says a converged carry-over entry lists fixes, because an `every fix worse` dispute leaves a finding the fixer agrees is real with no fix made.
Card 3 rewrites the rendered string, the const comment, the test row and the two loom stencil sentences, and says it greps for the old text.
A grep for the wording finds two more sites that make the same claim, in `contracts/stencils/claims_test.go`:
line 138 (`loom-rubric-plan-review.md` row, `is not part of the answer key`) has the why `a carry-over entry in the record's Open risks is an unreviewed fix, not a requirement the plan must carry`,
and line 171 (`loom-template-rework.md` row, `never plan an entry as scope`) has the why `a carry-over entry in the record's Open risks lists unreviewed fixes, never scope for a rework generation`.
No card edits either why.
Card 1 targets `contracts/stencils#wordingClaims`, but its Intent limits the edits to the two burler rows.
After the plan runs, the claims table would still say that every entry is a fix, the statement D7's acceptance criterion rules out for converged text.

**Fix:**
Add `contracts/stencils#wordingClaims` to Card 3's `**Edit:**` targets.
Card 1 already edits that target earlier in the plan, so the order rule still holds.
In Card 3's Intent, reword the two whys so they describe an entry's findings as fixed or disputed and unseen by a fresh reviewer.
The `must` phrases stay unchanged.

### [LOW] Card 1 cites this task's decision record as claim provenance

**Location:** `_lyx/plan/01-fixer-dispositions.md` (the `wordingClaims` paragraph: "Each claim's `why` names D1 to D3 of this task's decision record as provenance.")

**Issue:**
The `claims_test.go` header names the kinds of provenance a why carries: crucible round, finding, sabotage proof, live observation.
It also says a claim that cites one is protected and is never dropped.
`_lyx/discussion/decision-record.md` is not tracked in the code repository (`git ls-files _lyx` is empty), so after landing, "D2 of this task's decision record" points at nothing a later reader of `claims_test.go` can open.
It would still mark those claims as protected.
No other why in the table cites a decision-record entry, and D6 asks only for the claims themselves, not for this provenance.

**Fix:**
Drop the sentence from Card 1's Intent.
Each new why then states the rule its phrase pins, as the neighbouring rows do (for example `every change that addresses the finding must make the target worse before it is disputed`).

### [LOW] The one-rule-many-sites Shared Decision has the orchestrator state the rule, which Card 1 forbids

**Location:** `_lyx/plan/00-overview.md`, `### Decision: one-rule-many-sites`

**Issue:**
The Shared Decision lists "the orchestrator's Validate step" among the sites that "state or pin that text rather than its own variant".
Card 1's Intent says the Validate step "restates none of instruction 3's rules (`PATTERN-producer-pointer-rule`)".
It also says the step must keep the phrase `not whether it gets fixed` out of the orchestrator, because `TestComposePrompt_MinimalProfile` asserts that.
The decision record's constraint is the same: the fix orchestrator points at instruction 3 for the disposition rules and does not restate them.
An executor reading the Shared Decision alongside Card 1 gets two instructions for the same site.

**Fix:**
Reword the Shared Decision so the orchestrator's Validate step names the two checks and points at instruction 3.
Only the claims table, the pattern background, the overview line and the burlerengine package doc state or pin the rule.

## Focus verdicts

- Fidelity: D1 to D7 and every Constraint are carried. D1 to D3 and D6's stencil, claims, pattern and overview parts are in Card 1. D4 and D6's `doc.go` part are in Card 2. D7 is in Card 3. D5's no-change scope holds, since no card targets `bouncer-template-judge.md`. The Part B constraint on a D3 own change needs no card: `pattern-directive-review-fix.md` binds every fixer fix, and an own change is a fix. Apart from F2's provenance sentence, no card adds unlicensed scope.
- Card 1's Intent states the full bound on the `every fix worse` ground: changes outside the fixer's write surface count, an off-surface-only fix is deferred, a `Do not flag` finding is `premise false`, the evidence kinds are listed, the still-forbidden grounds are listed, and each `Disputed` entry names its ground. It also states the bound on the fixed-differently mark: name the proposed change not made and why, use the same evidence kinds, need no mark for a vague `**Fix:**` line, and stay on the write surface. Nothing is weakened.
- Card order: Card 2 uses `burler-step-3-fix.md`, which the earlier Card 1 targets, and Card 3 is independent, so no card depends on a later one. The Shared Decision is consistent with every card except for the orchestrator wording in F3.
- Test economy: `TestStencilClaims` (the `wordingClaims` table), the `prior round` subtest of `TestComposePrompt_PriorRounds` (which already asserts `deferred and disputed items`) and the `converged` row of `TestWriteCarryOver_RendersConvergedAndAcceptedEntries` all exist and cover the changed behavior. No card plans a new top-level test.
- Verify coverage: the repository has one module apart from a `testdata` fixture, so `go test ./...` and `go test -tags integration ./...` cover `contracts/stencils`, `internal/burlerengine` and `internal/discussionparser`. The section follows the spec's order.
- Member glyphs: Card 1's Go target is the `wordingClaims` member glyph. Card 2's `doc.go` is a package doc file with no symbol to name. Card 3 names `renderCarryOver`, `CarryOverConverged` and the test as members. No signature changes, so no re-sign arrow is required.
