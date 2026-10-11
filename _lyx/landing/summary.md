# Burler fixer weighs each finding before it acts

A Burler fixer now checks each finding's premise and, separately, the change the review's `**Fix:**` line proposes.
Before, `burler-step-3-fix.md` allowed a dispute only for a factually false premise and said nothing about a wrong proposed change, so a fixer applied a bad prescription because it had no other sanctioned outcome.

## What changes

The fixer decides each finding in this order:

1. **Dispute**, on one of two named grounds.
   `premise false` is today's ground.
   `every fix worse` is new: the premise holds, but the fixer shows with cited evidence (code, the fasit, the rubric, a `PATTERN` entry, a test or caller the change would break, or verify output) that every change addressing it makes the target worse.
   A finding the rubric's `Do not flag` list excludes is disputed on `premise false`.
   Severity, size, cost, taste and disagreement with the rubric or PATTERN still never justify a dispute.
   Each `Disputed` entry names its ground, the finding id and the evidence.
2. **Fix with own change**, when the premise holds but the proposed change is wrong or harmful and another change resolves the premise.
   The fixer-report's list of changes marks the finding as fixed differently, naming the proposed change it did not make and why.
   A fix that would need to leave the fixer's write surface is deferred, never disputed.
3. **Fix as proposed**, otherwise.

Supporting changes:

- The Validate step of `burler-template-fix-orchestrator.md` names both checks and points at instruction 3 for the rules.
- `priorRoundsBlock` in `internal/burlerengine/prompt.go` tells the next reviewer that a re-raised, previously disputed finding must state in its `**Issue:**` why the dispute's evidence does not hold.
  The next fixer can then concede or dispute again on new evidence.
- The converged carry-over entry from `renderCarryOver` in `internal/discussionparser/carryover.go` no longer says the fixer addressed each finding.
  It says each finding was fixed or disputed, that no fresh reviewer has seen the result, and that the fixer report says which.
  An `every fix worse` dispute leaves a defect nobody fixed, so the old text would have misstated it.
  `loom-template-plan.md` and `loom-template-rework.md` describe an entry the same way.
- `pattern/PATTERN-review-round.md`, `internal/burlerengine/doc.go` and `docs/overview.md` state both dispute grounds and the fix-with-own-change outcome.
- `contracts/stencils/claims_test.go` drops the `only for a factually wrong premise` claim and pins both grounds, the every-fix bound, the still-forbidden grounds, the fixed-differently mark and the Validate step's two checks.
  The prior-rounds assertion in `prompt_test.go` and the converged row in `carryover_test.go` follow the new text.

## Design decisions

- The judge is unchanged: stencil, Go-held decision rule and facts file.
  It never reads a fixer-report.
  A persisting dispute is ruled on by the finding's class and severity when a later review re-raises it, and a dispute never converges a segment on its own.
- The fixer-report stays free-form prose, existence-checked only.
  The fixed-differently mark is prose inside the existing changes list, not a new section, and no Go code parses it.
- No stencil marker is added or removed.

## To check

- In the round a segment converges, no fresh reviewer reads a dispute or a fixer's own change, so the fixer's cited evidence is the only bound on either.
  A MEDIUM or BLOCKING closing-round dispute reaches `## Open risks` with a pointer to the fixer report; a LOW or NIT one is recorded only in that fixer-report.
- A fixer could overuse the own-change disposition to swap a correct prescription for one it prefers.
  The next fresh reviewer is the only check, as for every fix today.
- The stencils, `claims_test.go` and the burler, discussionparser and shedadapters packages passed `lyx gate test`, and `lyx loom lint-comments` and `lyx loom lint-pattern` passed.
  No fixer was run by hand against a real review with a wrong `**Fix:**` line, so the new wording is untested against a live model.
- `burler-multi-llm` rewrites the round's wiring; if it renames or merges these stencils, the claims added here move with the text.
