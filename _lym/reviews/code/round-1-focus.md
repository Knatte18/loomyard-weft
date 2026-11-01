---
round: 1
focus:
  - "tools/tokencount/count.go, segment.go, shed.go: with no slugs, does a darn run now count, and does it count darn's own segments (writer, Code-Review rounds, landing) rather than tapestry segment names? Does a tapestry run still count the same as before?"
  - "tools/tokencount/recipes.go, main.go: does the recipe comparison give cost per run and per changed line, grouped by recipe over the same period? Check the zero-line and zero-run divisions, a recipe with no runs, and an unknown recipe name."
  - "tools/tokencount/landed.go and internal/summaryparser: where does the changed-line count come from, and does it handle a missing, malformed or archived-revision summary without miscounting or failing the whole report? Does the summaryparser change keep its other callers working?"
  - "contracts/specs/loom-final-summary-spec.md and the summaryparser docs: does the spec edit match the parser's behavior?"
  - "Test coverage of the added behavior (darn counting, recipe comparison, landed lines): is anything added without a test, or does a new test repeat an existing one in count_test.go, landed_test.go or time_integration_test.go?"
  - "tokencount.sh and tokencount.cmd: do the two wrappers stay in step, and does any usage or flag text change need a matching doc update?"
---

The decision record holds only the board entry under `## Goal`, so the entry is the measuring stick: count darn runs with darn's own segments, and compare recipes by cost per run and per changed line.
