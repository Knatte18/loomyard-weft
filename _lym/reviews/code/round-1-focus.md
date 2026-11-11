---
round: 1
focus:
  - "internal/heddleengine/store.go: does the unversioned-heddle.json migration turn claude-file into agents-file exactly once, leave versioned files untouched, and keep Save writing the version? Check partial or malformed files and a file already holding codex-file or agents-file."
  - "internal/heddleengine/render.go and target.go: does each of claude-file, codex-file and agents-file reach exactly the files the goal names (CLAUDE.local.md, AGENTS.override.md, both), including the claude directory CLAUDE.md case from the last commit? Check that no render path or cleanup of stale files still treats claude-file as both."
  - "internal/heddlecli/verbs.go and internal/heddleengine/overview.go: do `lym heddle set --help`, `lym heddle list` and HEDDLE.md state which files each target renders to, as the goal requires? Check that every place the target names are validated or listed knows all three."
  - "Scope against the goal: does any file or behavior change go beyond the three targets, the migration and the three documentation surfaces? Does any consumer of the old claude-file meaning remain unmigrated?"
  - "Tests in render_test.go, store_test.go, overview_test.go and cli_integration_test.go: is each new behavior covered, and does any new test duplicate an existing one or assert static help or doc prose?"
---

The decision record holds only the board entry under `## Goal`, so it is the measuring stick, and its other sections are empty.
The change description was not named in the prompt.
