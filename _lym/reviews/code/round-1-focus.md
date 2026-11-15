---
round: 1
focus:
  - "contracts/stencils/darn/darn-fragment-make-change-gated.md item 3: does the new wording let a gated writer run one named test through `lym gate test` before diagnosing, as the board entry asks, without opening a loophole for further gate runs by the writer or its subagents (item 2 still tells subagents to run none)?"
  - "The same item says 'Step 3 settles it' for an entry that landed work overtook: check in the darn writer template (darn-template-write and the other darn stencils) that a Step 3 exists and does settle an overtaken entry, so the pointer is not dangling."
  - "The instruction 'name its package and add `-- -run <TestName>`, following the gate rule from your system prompt': check it matches the actual `lym gate test` argument syntax and gate rule, so the writer can run the command as written."
  - "internal/darnengine/spec.go DarnTable doc comment: after the edit, does 'They act only on what the verify gate's fixer leaves' still have a clear antecedent, and do the GateFixed field comment, doc.go and stencils.go comments agree with the stencil on the exception?"
  - "Tests: does TestDarnSpec pin the gated and ungated prompts correctly (gatedReproduceRun present only when GateFixed), and is the TestWire_DarnRecipe constant change a needed sync rather than redundant coverage? Check for any other test or stencil still holding the old 'Run no `lym gate test` yourself.' string."
---

Seed focus from the diff: a stencil wording change in the gated darn fragment, mirrored in Go comments and two tests.
