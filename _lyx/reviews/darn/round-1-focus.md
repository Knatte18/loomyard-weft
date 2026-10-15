---
round: 1
focus:
  - "internal/orchengine/watcher.go `initCursor` and `typeStartColor`: does the pending decision compare the recorded strand and `LaunchedAt` against the bound session, and does a new or adopted session still get its color?"
  - "Every place a color typing lands (start color and reload color step): is `color_strand` and `color_launched_at` recorded only on a landed typing, never on a held or failed one?"
  - "The `State` struct and its persistence: do the new fields round-trip through the state file, and does a state file without them behave as the summary claims?"
  - "The `step=start-color` `orch: injection typed` log line: does it match the reload steps' format and fire only when the typing lands?"
  - "Tests in `internal/orchengine/watcher_test.go`: does each behavior the diff adds have coverage, and does any changed or new test duplicate an existing test's coverage? Check the budget row in `cmd/lyx/testdata/test-budget.yaml`."
  - "Docs: does the color paragraph in `internal/orchengine/doc.go` match the code, and does the change need a `PATTERN.md` or `docs/` update under CLAUDE.md's docs rule?"
---

Seeded from the board entry and the change description; the focus lists where to look, not verdicts.
