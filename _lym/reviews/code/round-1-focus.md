---
round: 1
focus:
  - "internal/consoletitle/consoletitle.go and internal/boardengine/handles.go: does every title come out as `<lead> [<hub>:<project>] <name>`, with a task as `T2 [ly:P1] slug`, a task under root as `T5 [ly:P0] slug`, and an orch leading with `ORCH`?"
  - "Orch titles: does the prime's orch read `ORCH [ly:P0] root` and a beam's orch `ORCH [ly:P1] <slug>`, with the last part naming the project (`root`), never the worktree (`prime`)?"
  - "Does any remaining caller, doc comment or prose (docs/overview.md, doc.go files, battenshed template.yaml, loomcli, reedcli/window.go, ideengine/spawn.go, fabricengine/hubworkspace.go) still describe or build the old `<hub>: <P#>: <T#>: <slug>` form?"
  - "The Slack channel names in `slack-per-orch-activation`: do they follow the same orch rule as the window titles, and does the diff touch them where the board entry asks?"
  - "Edge cases of the title builder: empty or missing handle, project or slug, and a task with no project, where a bracket or separator could be left dangling."
  - "Test changes in boardengine/handles_test.go, consoletitle_test.go and the integration tests: do the changed expectations match the new format, and does any new test duplicate an existing one?"
---

Focus covers the format change's core builders, the orch and root-project cases the board entry names, leftover old-format references, and test economy.
