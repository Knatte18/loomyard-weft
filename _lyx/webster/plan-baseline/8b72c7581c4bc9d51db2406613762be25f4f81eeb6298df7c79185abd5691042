# Card 1 — fixer-dispositions

**Edit:**
- `contracts/stencils/burler/burler-step-3-fix.md`
- `contracts/stencils/burler/burler-template-fix-orchestrator.md`
- `contracts/stencils#wordingClaims`
- `pattern/PATTERN-review-round.md`
- `docs/overview.md`

**Intent:**
Rewrite the fix-everything section of `burler-step-3-fix.md` so the fixer weighs each finding and gives it one of three dispositions, decided in D1's order:

1. Dispute, on one of two named grounds: `premise false` (the premise is false against the code or the fasit, today's ground) or `every fix worse` (the premise holds, but every change that addresses it makes the target worse).
2. Fix with own change: the premise holds, the review's `**Fix:**` line names a concrete change the fixer judges wrong or harmful, and some other change resolves the premise.
3. Fix as proposed: otherwise.

State D2's bound on the new ground: "every change" includes changes outside the fixer's write surface, so a finding whose only sound change leaves the surface is deferred with that reason, never disputed; the evidence is cited text (code, the fasit, the rubric, a `PATTERN` entry, a test or caller the change would break, or verify output); a finding the rubric's `Do not flag` list excludes is disputed as `premise false`, never on the new ground; severity, size, cost, taste, and disagreement with the rubric or PATTERN still never justify a dispute.
Each `Disputed` entry names its ground, the finding id and the evidence.
Keep the sentence that a dispute never converges a segment on its own out of this stencil, as today; the bound stays in the pattern background.

State D3's mark: a finding fixed with the fixer's own change is marked as fixed differently inside the fixer-report's existing list of changes, naming the proposed change it did not make and why, with the same kinds of evidence; a finding whose `**Fix:**` line is vague or absent is fixed as the fixer sees fit and needs no mark; an own change that would need to leave the fix-scope write surface is deferred with that reason.
The deferred section keeps its meaning and is not a fourth disposition for disagreement.
Keep the exact phrases the claims table still pins: `` `Disputed` section ``, `the finding id and the evidence`, `not whether it gets fixed`, `never push`, `nothing fixed`, `adds a test only when the finding is a coverage gap`.
Drop the sentence `A dispute is allowed only for a factually wrong premise;` and its continuation; no other file carries that text.
The marker set stays `fix_scope_rules`, `review_path`, `fixer_report_path`, with no conditional, so the header comment's "all three" still holds; keep the fixer-report free-form, with no new section.

In `burler-template-fix-orchestrator.md`, step 3 (Validate) checks each finding's premise against the code and the fasit and, separately, the change its `**Fix:**` line proposes, and points at instruction 3 (`{{.instruction_3_path}}`) for what to do with each result; it restates none of instruction 3's rules (`PATTERN-producer-pointer-rule`), and in particular never carries the phrase `not whether it gets fixed`, which `TestComposePrompt_MinimalProfile` asserts stays out of the orchestrator.
The marker set and the header comment's closing sentence are unchanged.

In `wordingClaims`, in the `burler-step-3-fix.md` row, replace the `only for a factually wrong premise` claim with claims that pin: the `premise false` ground, the `every fix worse` ground, the every-fix bound (a phrase stating that one sound change means the finding is fixed, not disputed), the still-forbidden grounds (severity, size, cost, taste and rubric disagreement never justify a dispute), the ground named per `Disputed` entry, and the fixed-differently mark inside the changes list.
In the `burler-template-fix-orchestrator.md` row, add a claim that Validate checks each finding's premise and its proposed change.
Each claim's `why` states the rule its phrase pins, as the neighbouring rows do.
Every claim is a short distinctive phrase copied from the stencil text as written on this card.

In `pattern/PATTERN-review-round.md`, the opening sentence and the dispute bullet state both dispute grounds, the every-fix bound and the fix-with-own-change outcome, and keep that a dispute never converges a segment on its own and that severity, size, cost or disagreement with the rubric never justify one.
In `docs/overview.md`, the burler line's sentence on the fixer says it disputes a finding only with evidence that its premise is false or that every fix makes the target worse, and may fix a real finding with its own change when the proposed one is wrong.
The `PATTERN.md` entry line stays as written.
Every markdown edit uses semantic line breaks.

**ImpactSummary:** Two burler stencils, their claims rows, one pattern background and one overview line; no Go production code, no marker change, and the hub copies refresh through the existing stencil reconcile.

**Commit:** 1: Fixer dispositions: two dispute grounds and fix with own change
