Lens fix-code-correctness: F2 and F3 hold; the F4 `show` assertion passes with the defect in place.

**LOW** — `internal/heddlecli/cli_integration_test.go`, `RenderWritesFileTargetsAtTheAnchor`: the `show claude-file` assertion does not depend on the anchor.
- It checks only exit code 0 and that the output contains "text of rules", which the stored entry gives at any directory.
- The render assertions fail on a revert of `verbs.go:295`; the show half would not on a revert of `verbs.go:335`.

**F2** `anchored_hub_wiring_seeds_the_lines_under_the_anchor`: no finding. With `"."` passed in place of `l.AnchorRel`, the `/backend…` counts would be 0 and the root counts 1. The row passed in the narrowed gate run.

**F3** help text and package doc: no finding. String and comment edits; the `heddlecli` package passed.

Callers of changed signatures: no finding, no signature changed.

Tool calls:
1. grep of `func (r Renderer) Show` in `internal/heddleengine`: to see how `Show` uses `anchor`, which the context lacked.
