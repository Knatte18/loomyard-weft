---
round: 1
focus:
  - "D4 and D5: verify against the code that block keys carry an enclosing function name and that go/parser naming matches it for methods, closures and generics; a false premise here breaks selection."
  - "D1 and D5: the attack-surface bound for tests a coverage scope skips. Check each skip path (importing package, test absent from map, tier unmapped, deleted file) states what it lets through and what bounds it."
  - "D4: a change in a changed package that is not a function body, such as a package-level var, const or type, or a method on a changed type. Does D5 widen for each, or can a selected set miss tests that reach it?"
  - "D2 and D3: the spawn-scan verdict and out-of-process or unresolved tests. Check the record states where the verdict comes from and that nothing is left for Plan to discover."
  - "Open risks: the two questions for the parent. Check whether an Added after Discussion entry or a board note titled Decision for gate-scope-by-coverage-map already answers either, and whether each is a decision-class question or a choice the writer can make."
  - "D6: the command budget and the green-entry `-run` recording. Check the record points to a settled fact and defers no lookup to the implementer."
  - "Leanness: Constraints, Auto-mode assumptions and Open risks for content that belongs in support-log.md or restates a Decision; apply the completeness-before-leanness test before any relocation finding."
  - "Acceptance criteria: each D1 to D8 id appears as a whole word in a bullet, and each criterion is checkable behavior or a Review: item."
---

Seeded from a read of both artifacts and the discussion spec.
The writer was not told to gather rejected alternatives in the record, so none is flagged as missing.
