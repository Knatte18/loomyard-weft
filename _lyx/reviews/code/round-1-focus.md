---
round: 1
focus:
  - "pattern/PATTERN-test-isolation.md: the parent's main changes since 93f1c8942 (`git diff 93f1c8942e059ae8e8adcea22f0345f1035daae9 main -- pattern/PATTERN-test-isolation.md`) must be carried into the merged survivor, which absorbs testkit and hubforge-fixtures; check no parent rule was lost or contradicted in the merge."
  - "PATTERN.md and the `test-isolation` entry: after the merge-in, does the entry's text and its `pattern/` link still match the merged file, and does `internal/pattern.Check` still see every `pattern/` file named by exactly one entry?"
  - "References to `PATTERN-testkit` and `PATTERN-hubforge-fixtures` IDs: does any `.go` file, contract spec or message touched or added by the parent's change cite an absorbed ID instead of `PATTERN-test-isolation`?"
---

Amendment round: only the parent's change to `pattern/PATTERN-test-isolation.md` and its alignment with the board entry's merge group `test-isolation` are in scope.
