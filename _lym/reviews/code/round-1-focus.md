---
round: 1
focus:
  - "internal/boardengine/render_test.go: confirm the kept tests (RenderToDisk in-place rewrite, output path, manifest cleanup, TestFormatAge) still assert the behaviour their names state, and that nothing other code relies on lost its only coverage in the deletions."
  - "internal/boardengine/view_test.go: check the literal board-data.js placement and section-list assertions cover every entry placement the old heading parsing covered, and that board-data.js holding each entry (read by board.html) is still asserted."
  - "internal/boardcli/text_test.go: settle whether TestRenderCompact pinned layout alone, or also a behaviour other code relies on (filtering, ordering, which entries appear) that now has no test."
  - "Scope against the answer key: the record asks to sweep the repository for other human-read renderers carrying layout goldens; check whether any remain outside the four touched files, and that the diff changes no production code."
  - "internal/boardengine/layer_test.go: check the one-line change is within the scope of deleting layout goldens."
  - "Loosened checks: any assertion in a test that exists at the review base that the diff widens or removes beyond the layout goldens, with no Loosened-Check trailer."
---

Diff is deletion-heavy and touches tests only; the main risk is losing coverage of behaviour that the record says to keep.
