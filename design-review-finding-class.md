From `manifest/designs/review-finding-classification.md`; its item 5 (matching "do not flag" instructions on writer and reviewer) has shipped.

- Add a finding-class field (`design`, `scope`, `decision`, `consistency`) to burler's review contract and the rubrics; `internal/burlerengine/verdict.go` has only severity today.
- End the round loop when a round returns zero gating-class findings, keeping the cap as a backstop.
- Emit per-class counts in the round record, so the same class recurring across rounds is visible.
- Add a line to the Review Round Invariant: class decides who decides and when the loop stops, never whether a finding is fixed.
