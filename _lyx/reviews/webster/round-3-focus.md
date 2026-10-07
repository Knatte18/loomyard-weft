---
round: 3
focus:
  - "Every Go comment line the diff adds or changes, including the fix commits: does each break fall at a sentence end, a semicolon, or a comma before a coordinating conjunction that starts an independent clause, with one sentence per line? Scan all of them, since the lint can miss a paragraph that sits in a block holding a heading, list or code line."
  - "internal/commentlint/lint.go isProseBlock and its rows: does a block judge each paragraph on its own, skipping only a paragraph that holds code, a list item or a heading, and does a row cover a wrapped paragraph beside such a paragraph?"
  - "Every test the diff adds or changes, in each package it touches: does it call t.Parallel, in its subtests too, unless a comment names the global state that prevents it, and does it cover something no existing test covers at the public surface?"
  - "internal/impactset/guards.go and the round gate in internal/loomshed/verifygate.go: does a guard marker in a tmux or llm file produce an error naming file and line, and do the guard tests assert the line for both rows?"
---

Round 3 targets the places round 2 found defects.
Each entry names where to look and the question to settle, and sets no severity.
