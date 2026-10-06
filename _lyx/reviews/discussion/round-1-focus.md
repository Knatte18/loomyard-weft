---
round: 1
focus:
  - "Decisions 9, 11 and 12 name `cmd/testtiming` flags (`-redundancy`, `-tags`, `-pkg`, `-out`) and a section layout in `docs/benchmarks/test-redundancy.md`: settle whether the tool at this task's base commit supports each flag and a section-scoped replacement as stated, or whether the decision rests on a false premise."
  - "Decision 10 and the first acceptance criterion depend on `test-suite-prune-core`'s D2 spelling of `//testtiming:keep <reason>`: settle whether the record states enough for the plan writer to produce that exact spelling, and whether the criterion is checkable at this task's base when the tool ignores the comment."
  - "Decisions 2, 3 and 5 each let a test be kept, moved or left in place with a 'stated reason': settle whether every such path says where the reason lives and what bounds it, so that Decision 2's move-then-delete path cannot drop an assertion unnoticed."
  - "Decisions 1, 5 and 7 against the core boundary: settle whether a deletion, fold or reference update in an in-scope package can force an edit to a core file or `internal/testkit/`, and whether the record says what happens then."
  - "Decision 8 and Decision 12 as a pair: settle whether re-running a `failed` test alone, and the coverage-drop rule, are consistent with Decisions 2 and 4, given that coverage does not see a lost assertion."
  - "Acceptance criteria against Decisions: settle whether every Decision's observable outcome has a matching criterion and no criterion contradicts a Decision, including the `go vet` tag matrix and the `llm` exclusion."
---

Seeded from the decision record and support log.
No entry targets the three rubric `Do not flag` items or content the discussion stencil tells the writer not to gather (exact signatures, `file:line` citations, exhaustive pattern research).
