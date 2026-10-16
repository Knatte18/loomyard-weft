---
round: 1
focus:
  - "tools/sandbox/SANDBOX-CREEL-SUITE.md against the moved scenarios in the old SANDBOX-SHUTTLE-SUITE.md: do S9 and C1 to C13 keep their ids and wording, and does the file stand alone with no dangling reference to the shuttle suite?"
  - "tools/sandbox/SANDBOX-SHUTTLE-SUITE.md after the cut: does it keep S1 to S8 intact, with no leftover reference to a moved scenario?"
  - "tools/sandbox/main.go and suite.go: does the `creel-suite` subcommand embed `creelSuite` and dispatch through the shared `runSuite`, and does the usage text and error path name it correctly?"
  - "tools/sandbox/main_test.go and suite_test.go: do the new table rows cover the new behavior without duplicating an existing test, and does any package stay within its row in cmd/lyx/testdata/test-budget.yaml?"
  - "sandbox/posix/creel-suite.sh and sandbox/win/creel-suite.cmd against the existing shuttle-suite launchers: same conventions, executable bit, line endings and argument passing?"
  - "docs/sandbox-howto.md: does it name the creel suite and its launchers accurately, in semantic line breaks, and are the docs-land-in-same-commit duties met, including the module doc in the sandbox package's doc.go if it lists subcommands?"
  - "The C13 board note: was it filed through the board as the card asked?"
---

Seed focus from the plan's three cards and the diffstat.
