---
round: 1
focus:
  - "internal/boardengine/render.go `collapsibleProjectSection`: check the emitted HTML against the Goal. A blank line must follow `<summary>` and precede `</details>`, the section must be `<details open>`, and the summary must carry the handle, title, link to the project file and open-member count. Settle whether the markdown inside renders in VS Code Markdown Preview and on GitHub."
  - "Settle whether the summary text escaping is correct and equivalent to the old heading: `html.EscapeString` replaces `cellText`'s pipe escaping, and an HTML link inside `<summary>` with `<strong>` wrapping must not change what a handle or title containing `&`, `<`, `\"` or `|` shows."
  - "Check the refactor of `projectSection` into `openMembers` and `projectSectionContent`, and `projectLink` into `projectFile`. Settle whether project files, nested sub-project sections, tables and design files render byte-identically to before (the Goal's Limits), including the root's member count rule that excludes its beside-it projects."
  - "Check scope: the diff must change only the README's top-level project sections and the root's section. Settle whether nested sub-project sections in the README still render through the old heading path as the commit message claims."
  - "Check the edits in docs/overview.md and internal/boardengine/doc.go against the diff's behavior, in particular the claims that the summary stands in for a `##` heading and that the contents keep their levels below `##`."
  - "Check render_test.go for test economy: tests that duplicate existing coverage, per-helper tests the public render tests already cover, and any behavior the diff adds with no test (blank line placement, open attribute, escaping, root section)."
---

The decision record holds only the board entry under `## Goal`, so that entry is the measuring stick.
