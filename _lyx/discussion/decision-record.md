# Decision record — fixer-critical-of-findings

## Goal

A Burler fixer weighs each finding before it acts: it checks the finding's premise and, separately, the change the review's `**Fix:**` line proposes.
It may dispute a finding whose every fix would make the target worse, and it may fix a finding with its own change when the premise holds but the proposed change is wrong.
Today `contracts/stencils/burler/burler-step-3-fix.md` allows a dispute only for a factually false premise, and says nothing about a wrong proposed change, so a fixer applies a bad prescription because it has no other sanctioned outcome.

## Scope

In:

- The fixer's disposition rules in `contracts/stencils/burler/burler-step-3-fix.md`, and the fixer-report contents they require.
- The Validate step of `contracts/stencils/burler/burler-template-fix-orchestrator.md`, which names the two checks without restating instruction 3's rules.
- The next-round reviewer's prior-rounds instruction in `internal/burlerengine/prompt.go` (`priorRoundsBlock`): a re-raised disputed finding answers the dispute's evidence.
- The pinned claims for both stencils in `contracts/stencils/claims_test.go`, and the prior-rounds assertion in `internal/burlerengine/prompt_test.go`.
- Docs that state the dispute rule: `internal/burlerengine/doc.go`, `pattern/PATTERN-review-round.md`, `docs/overview.md`.

Out:

- The judge: `contracts/stencils/bouncer/bouncer-template-judge.md`, the Go-held decision rule and the facts file are unchanged; the judge still never reads a fixer-report.
- The review-file format and the reviewer's own review step (`burler-step-2-review.md`).
- The carry-over entry text in `internal/discussionparser/carryover.go`.
- Burler's two-agent wiring, which `burler-multi-llm` replaces; this task changes stencil text and one prompt string that move with the stencils.
- Any Go parsing of the fixer-report; it stays free-form prose, existence-checked only.

## Decisions

### D1 — Every finding gets one of three dispositions

Decision: for each finding the fixer decides, in this order:

1. **Dispute** — the premise is false against the code or the fasit (today's ground), or the premise holds but every change that addresses it makes the target worse (new ground, D2).
2. **Fix with own change** — the premise holds and the review's proposed change is wrong or makes the target worse, but some other change resolves the premise (D3).
3. **Fix as proposed** — otherwise.

The deferred section keeps its meaning (an operator decision on a real tradeoff, or a capability the fixer lacks) and is not a fourth disposition for disagreement.

Rationale: the brief's two additions are distinct cases.
A wrong prescription on a real defect still needs a fix, so it is never a dispute; only a finding no change can improve is disputed.
Ordering the checks keeps "fix with own change" from becoming a softer dispute.

### D2 — The new dispute ground and its bound

Decision: a finding whose premise holds may be disputed only when the fixer shows that every change addressing it makes the target worse, not only the proposed one.
The evidence is cited text: code, the fasit, the rubric (its `Do not flag` list included), a `PATTERN` entry, or a test or caller the change would break, or verify output.
Severity, size, cost, taste, and disagreement with the rubric or PATTERN still never justify a dispute.
Each `Disputed` entry names its ground (`premise false` or `every fix worse`), the finding id and the evidence.

Bound: the new ground lets a finding stay unfixed only when cited evidence shows harm from every addressing change; a finding where one sound change exists falls to D3 and is fixed.
A dispute still never converges a segment on its own (D5).

Rationale: an example the old rule cannot express is a true duplication whose only removal is an import a leaf-package rule forbids; the finding is real and no fix is legal.
Requiring evidence against every fix keeps the ground from absorbing ordinary disagreement.

### D3 — Fix with own change when the prescription is wrong

Decision: when the premise holds and the review's `**Fix:**` line names a concrete change the fixer judges wrong or harmful, the fixer makes a different change that resolves the premise.
The fixer-report's list of changes marks that finding as fixed differently, naming the proposed change it did not make and why, with the same kinds of evidence as D2.
A finding whose `**Fix:**` line is vague or absent is fixed as the fixer sees fit and needs no such mark.
The fixer's own change stays inside its fix-scope write surface; a change that would need to leave it is deferred with that reason.

Bound: this lets the fixer skip the reviewer's prescription, never the finding; the premise is still resolved this round, and the next round's fresh reviewer judges the result.

Rationale: the fixer has read the code and the fasit as closely as the reviewer, and `loom.yaml` can give it a stronger model than the reviewer's; a fixer that cannot overrule a bad prescription spends that judgment applying it.
Recording the departure inline keeps the fixer-report's sections as they are while giving the next reviewer the reason.

### D4 — The next reviewer answers a dispute's evidence

Decision: `priorRoundsBlock`'s closing sentence gains one rule: a reviewer that re-raises a finding a prior fixer-report disputed states, in that finding's `**Issue:**`, why the dispute's evidence does not hold.
The existing phrase `deferred and disputed items` stays.

Rationale: today a dispute is settled by the next round's fresh reviewer, who re-raises or drops it, and the judge rules on the resulting review.
That path stays, but a reviewer that re-raises without engaging the evidence turns every dispute into a circling key; the rule makes a re-raise carry an answer the judge and the next fixer can read.

### D5 — The judge and convergence are unchanged

Decision: the judge stencil, the decision rule, the facts file and the carry-over text do not change.
A disputed finding's key stays open in the ledger while a later review re-raises it, and the existing CIRCLING path settles a dispute that persists.
A closing-round dispute reaches `## Open risks` like any other addressed finding, with the fixer-report path the carry-over entry already names.

Rationale: the brief says the judge rules on disputes as today; the convergence rules already treat a dispute as no convergence signal.

### D6 — Docs and claims follow the stencil

Decision: `pattern/PATTERN-review-round.md`'s opening sentence and its dispute bullet, the `internal/burlerengine/doc.go` paragraph on exceptions, and the `docs/overview.md` Burler line state both dispute grounds and the fix-with-own-change outcome.
The `PATTERN.md` one-liner (`fix or dispute each finding with evidence`) stays as written, since it still holds.
`contracts/stencils/claims_test.go` replaces the `only for a factually wrong premise` claim with claims for both grounds, the every-fix bound, the still-forbidden grounds and the fixed-differently mark, and adds a fix-orchestrator claim that Validate checks the premise and the proposed change.

Rationale: `PATTERN-review-round` currently says a dispute needs evidence that the premise is false, which the new ground contradicts; the CLAUDE.md docs rule puts the change in the same commit.

## Constraints

- `PATTERN-review-round` is changed by this task: its background gains the second dispute ground and the fix-with-own-change outcome (D6); every other clause of it stands.
- `PATTERN-stencil-ownership`: stencils stay read at call time through `internal/stencilstore`; the hub copies refresh through its existing reconcile, with no new seeding path.
- `PATTERN-producer-pointer-rule`: the fix orchestrator points at instruction 3 for the disposition rules and does not restate them; the review format stays in the file `review_format_path` names.
- `PATTERN-test-economy`: the changes are pinned by extending the existing claims table and the existing prior-rounds assertion, with no new top-level test.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule apply to every edited Go comment and markdown file.
- The stencils keep their no-conditional, every-marker-required shape; no marker is added or removed.
- Part B of `pattern-directive-review-fix.md` (a fix must not introduce a constraint violation) applies unchanged to a D3 own change.

## Auto-mode assumptions

- The fixer-report stays free-form, so the fixed-differently mark is prose inside the existing changes list, not a new section; no Go reads it.
- The new dispute ground covers a premise that holds as fact but whose every remedy is harmful; a finding the rubric's `Do not flag` list excludes counts as a false premise, as today.
- The reviewer-side rule (D4) is in scope, though the brief names only the fixer, because without it the broader dispute ground feeds the CIRCLING path instead of settling.
- Model defaults and the judge's model are untouched; whether a cheaper fixer should keep these freedoms is not this task's question.

## Open risks

- A fixer may overuse D3 to swap a correct prescription for one it prefers; the only check is the next fresh reviewer, as for every fix today.
- A closing-round dispute converges without a fresh reviewer seeing its evidence; the carry-over entry's fixer-report path is the only pointer to it, as today.
- `burler-multi-llm` rewrites the round's wiring; if it renames or merges these stencils, the claims added here move with the text.

## Acceptance criteria

- `burler-step-3-fix.md` states the three dispositions in D1's order, both dispute grounds with D2's every-fix bound and evidence kinds, the still-forbidden grounds, the ground named per `Disputed` entry, and D3's fixed-differently mark inside the changes list.
- `burler-template-fix-orchestrator.md`'s Validate step checks each finding's premise and its proposed change, and points at instruction 3 for what to do with the result.
- The next reviewer's prior-rounds block carries D4's rule and still contains `deferred and disputed items`.
- The claims table pins: both dispute grounds; the every-fix bound; severity, size, cost and rubric disagreement still never justify a dispute; the fixed-differently mark; the Validate step's two checks. The retired `only for a factually wrong premise` claim is gone.
- A rendered fixer prompt (the existing compose test) still keeps the fix-everything body out of the orchestrator.
- `pattern/PATTERN-review-round.md`, `internal/burlerengine/doc.go` and `docs/overview.md` state the same rule as the stencil, and no file still says a dispute needs a false premise alone.
- The burler, stencils and shedadapters packages pass `lyx gate test`, and `lyx loom lint-comments` passes on edited Go files.

## Notes for the plan writer

- The judge stencil's absence from scope is deliberate (D5); its claims in `claims_test.go` stay as they are.
- `internal/burlerengine/engine_test.go` has a case whose fixer-report carries a `## Disputed` section; it tests completion, not wording, and needs no change.
- The `burler-step-3-fix.md` header comment counts its markers ("requires all three"); the marker set does not change.
