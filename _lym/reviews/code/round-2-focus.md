---
round: 2
focus:
  - "internal/boardengine/intake.go `issueContent` and its callers in `internal/boardcli/intake.go`: does the import of an issue that names `.lym/` evidence copy that evidence into the entry body or a durable file the entry references, on both the new-note and the fold path, or only list the paths? Judge against the board entry's first point, which asks to copy the evidence the issue names into the entry or into a file that outlives the source worktree."
  - "internal/boardengine/intake_test.go `TestImportIssueFold` and `TestImportIssueNewEntry`: do rows pin the folded body for a blank-body issue and for a comment naming a `.lym/` path, together with `EmptyBody` and `LocalPaths` on a fold's result?"
  - "contracts/stencils/friction/friction-template-reflection.md near the `--attach` paragraph, and `appendAttachments` in internal/selfreportcli/cli.go: do the stencil's rule that the issue body ends with a provenance line and the `--attach` advice agree on where attached sections land?"
  - "internal/selfreportcli/cli.go `appendAttachments` and `attachmentSection`: how are invalid UTF-8 content, a backtick in the base name, an empty `-b` with `--attach`, and the body-limit boundary of the whole rendered body in runes handled, and do the tests in internal/selfreportcli/cli_test.go pin them?"
  - "plugins/ly/skills/board/SKILL.md and the Long help of `intake import` in internal/boardcli/intake.go: does the wording about named `.lym` evidence match what an issue filed with `--attach` actually carries?"
---
