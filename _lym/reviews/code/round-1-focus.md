---
round: 1
focus:
  - "internal/reedengine/panebin.go, bindings.go, spawn.go, window.go, lock.go: does every production path still resolve the executable through os.Executable via the Engine field set in New, with no package-level executablePath left, and does an unresolvable path still log through logger.Warn and drop the prelude or binding?"
  - "internal/reedengine tests: does each new t.Parallel test avoid process-global state (t.Setenv, logcapture, shared files, working directory), and does each test left serial carry a comment naming that state?"
  - "internal/reedengine tier2 tests: do parallel tests share any tmux socket, temp directory or fixed name, and does any test now race on state a sibling mutates?"
  - "internal/reedengine/panebin_test.go, spawn_test.go, window_test.go, windoworder_test.go: do the injected-lookup tests still assert the pane launch line, window command, switch and follow commands carrying the path, and the dropped prelude or binding on an unresolvable path, with no assertion lost?"
  - "internal/battenrecipe/recipe_test.go: does the merged row-name test assert both the row order and NameRunShed's durable value Run-Shed, with the deleted test's only distinct assertion kept and 8 top-level tests remaining?"
  - "docs/benchmarks/test-suite-timing.md: does the correction note name the real counts for shell, boardtest and battenrecipe and the reedengine reading, and are the claims in it borne out by the source?"
  - "Scope: are internal/shell and internal/boardengine/boardtest tests untouched, and does the diff change anything outside the reedengine seam, the battenrecipe fold and the audit note?"
  - "Docs in the same commit: does the reedengine doc.go change match the seam move, and does any other doc or PATTERN text still name the package-level variable?"
  - "Test economy: do the reedengine test edits add tests or assertions that duplicate existing coverage?"
---

Seed focus from the board entry and decision record; the range is the three commits over `4d52e208`.
