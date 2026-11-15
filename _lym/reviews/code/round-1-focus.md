---
round: 1
focus:
  - "Scope against the board entry: the entry's second point, the markdown preview failing in a window lym opens, and its Idea to settle whether lym's settings are at fault, `render_markdown` and `workbench.editorAssociations` in `internal/vscode/config.go`. The diff touches only the template, its doc, the test and `docs/overview.md`. Settle whether the diff or `_lym/landing/summary.md` addresses that point, records a finding on it, or leaves it silent."
  - "Remaining references to the old default: grep the repo, `docs/`, `README.md`, stencils and other `internal/` packages for text or tests that still say a beam opens `README.md` by default, besides the three prose spots the diff already updates."
  - "`internal/ideconfig/template.yaml`: confirm `beam: []` still loads as an empty non-nil list, matching the `[]string{}` the test expects and the `prime: []` and `task: []` handling. Check that the unchanged merge of hub and local layers (a hub's `open_files.beam: []` over the template) behaves the same."
  - "`_lym/landing/summary.md` against the diff: a claim the diff does not bear out, including any claim about the preview association."
---

The diff is small and mechanical for point 1 of the board entry.
The open question is point 2, which the decision record's Goal states but the diff does not visibly touch.
