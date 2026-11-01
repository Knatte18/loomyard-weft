---
round: 1
focus:
  - "internal/boardengine/render.go `projectFacts`: confirm `b.out.DesignPrefix` is set for both README.md and project files, so the link target resolves from each file's directory and matches the `design-<slug>.md` file actually written for a project with a body."
  - "Settle whether a project's body is rendered to `design-<slug>.md` under exactly the condition `slugCell` tests (`Body != \"\"`); a mismatch would leave a dangling link or an unlinked readable body."
  - "internal/boardengine/render_test.go: the fixture adds a Body only to `ship`; check the same golden still pins the no-body project (`tidy`, nested `inner`) keeping a plain slug in README.md and its project file, as the Design bullet requires, and that no other existing test or golden renders a project with a body whose expected facts line is now stale."
  - "internal/boardengine/doc.go and the `projectFacts` and test-doc comments: check each claim against the diff and that other docs describing the facts line (grep `facts line`, `code span`) carry the same change."
---

Small, single-commit diff in `internal/boardengine`: `projectFacts` swaps `codeSpan(project.Slug)` for `slugCell(project, b.out.DesignPrefix)`.
The decision record holds only the board entry under `## Goal`, so it is the measuring stick as written: link in README.md and project files alike when the project has a body, plain slug otherwise.
