---
round: 2
focus:
  - "internal/gatecli/mutation.go `mutationStep.run`: does eligibility for the step branch on any language or scoper literal, or does it ask a row-keyed capability lookup, as PATTERN-code-conventions requires?"
  - "internal/gatecli/mutation.go `mutationStep.answer`: when `mutationCall.run` returns an error, can the written report still record Functions and FileHashes so that `MutationReportMatches` holds for HEAD?"
  - "internal/gatecli TestGateMutate report-step rows: is each judge-nothing branch of `mutationStep.run` and `answer` (red gate, no hub, no parent branch) asserted, and are the survivors, unjudged mutants and uncovered blocks written to mutation.json asserted?"
  - "internal/gateslot/doc.go: does the non-tier slot holder list and the bounds paragraphs for mutation_max_mutants and mutation_budget_min name the mutation report step and the even spread?"
---

Round 2 re-reviews the whole range, with the entries above as places to look first.
