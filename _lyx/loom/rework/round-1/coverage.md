# Rework coverage

- F1 (BLOCKING, the critical-fixer judge step renders at Discussion-Write against the record's superseding decision): card 4.
  The judge steps become two part stencils, `twill-judge-critical` and `twill-judge-plain`, which `Directive` chooses between through `Values.CriticalJudge`.
  Plan-Write sets the critical judge, and Discussion-Write gets back `main`'s judge steps with its rejection rule.
  The card also updates the markers, claims, twilldirective and Discussion-table tests, and reverts the Discussion wording in `docs/overview.md` and the `twilldirective` package doc.
  The D3 re-prompt rule and the D5 boilerplate stay at both hosts.
