---
round: 1
ledger:
  - key: card1-testloomrubrics-specs-dir-darn-review
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card1-testtiming-keep-sentence-misplaced
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings against card 1 (`_lyx/plan/01-replace-stencil-claims.md`).
F1 (BLOCKING, design): the prescribed `TestLoomRubrics` assertion that every rendered rubric contains `/abs/specs/dir` fails for the darn-review rubric, which carries no `specs_dir` marker; the review's fix scopes the assertion per row.
F2 (LOW, consistency): the `//testtiming:keep` sentence sits in the `TestLoomRubrics` paragraph but belongs to `TestStencils_TopLevelMarkers`.
Both fixes are local corrections whose result the finding text already fixes, so both are `mechanical`.
