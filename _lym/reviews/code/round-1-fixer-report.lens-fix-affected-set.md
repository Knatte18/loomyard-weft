```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the fix change set (rerun the searches for set-premised findings and check every member is covered).

**Unchecked findings:** none. F1 to F4 were each covered by one of the three searches (F2 and F4 share one).

**Findings**

- **LOW.** Changed text: the new "The measurement calls of an entry … fall outside these rules" rule in both fragments (F4's narrowing).
  - **Summary:** the sibling descriptions of this rule still say "the gate calls of an entry whose deliverable is gate measurements". That wording covers all of the entry's gate calls, including the per-commit run, which is the reading F4 removed.
  - **Evidence:**
    - `contracts/stencils/stencils.go:55` and `:64`
    - `internal/darnengine/spec.go:95`
    - `contracts/stencils/darn/darn-fragment-make-change.md:3` ("bar the gate calls of an entry…", the clearest case, because the fragment body now says the per-commit run still follows)
    - `contracts/stencils/darn/darn-fragment-make-change-gated.md:5`
  - **Proposed change:** say "measurement calls" at those five sites.

**Covered, no finding**

- **F1:** no `measurementEntryRule` or other assertion on the rule's wording remains in any Go file.
- **F2:** the old "With no parent, run none of them and name them in the change description" sentence is gone from both fragments. The remaining "name them in the change description" (`darn-fragment-make-change.md:18`) is the older clause about skipped tests, which the review excluded.
- **F3:** the ungated item 2 is the only darn site that hands subagents a gate rule, and it now keeps the measurement calls in the writer's session. The gated item 2 already forbids subagents any `lym gate test`.

**Tool calls (3)**

- `grep` over Go files for the rule's names and phrases, to find any test or code member outside the files shown (F1).
- `grep` over `contracts`, `internal`, `docs` and `pattern` for the no-parent clause and the rule's description phrases, to find twins that my context did not show together (F2, F4).
- `grep` for "subagent" in the darn stencils, to find other sites handing subagents gate rules (F3). There is no darn-named engine stencil file.
