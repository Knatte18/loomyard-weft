---
round: 1
focus:
  - "internal/vscode/config.go and internal/ideengine/spawn.go: does any remaining caller, doc comment or other doc (doc.go, docs/overview.md, PATTERN) still reference the deleted TaskChain variants, claudePath or the folderOpen chain? Check with grep across the repo."
  - "internal/vscode/config.go: does the single tasks.json task match the board entry (absolute stamped lyx path, no runOptions, no dependsOn), and is the tracked-tasks.json skip and overwrite behavior unchanged?"
  - "internal/vscode/config_test.go: 438 lines changed; does the rewrite drop coverage of behavior the diff keeps (settings.json keep, info/exclude handling, tracked-tasks.json skip), and do the new tests duplicate the ideengine integration assertions?"
  - "internal/ideengine integration tests: does the SeedTask assertion exercise the Seed-IDE path for real, and does every claim in the change description's Tests section hold against the diff?"
---

Seed focus from the change description and diffstat; no round has been judged yet.
