# Card 3 — Prune Burler's prompt tests to Go-built and Go-chosen text

**Edit:**
- `internal/burlerengine#TestComposePrompt_FocusDirective`
- `internal/burlerengine#TestComposePrompt_MinimalProfile`
- `internal/burlerengine#TestComposePrompt_ParentDirective`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/burlerengine#requireContains`

**Intent:**
Prune the Burler engine's assertions on stencil prose by D1, keeping its assertions on text `prompt.go` builds in Go (the fix-scope, tool-use, prior-rounds and cluster-rules blocks), which the decision record leaves out of scope:

- `TestComposePrompt_FocusDirective`: drop the "The rubric binds over the focus directive" assertion; the focus path (a Go-filled value) still tells the directive-present branch apart, and "Focus directive for this round" still tells the absent branch apart.
- `TestComposePrompt_MinimalProfile`: the seat-stencil exclusion loop keeps "SINGLE message" and "subagent_type", the Go-built cluster block's tokens, and drops "not whether it gets fixed", "verdict:", "findings:" and "lyx shuttle await", which are stencil prose.
  Its doc comment and `//testtiming:keep` reason say the seat stencils are checked to exclude the Go-built cluster block, not every downstream instruction body.
- `TestComposePrompt_ParentDirective`: replace the full operator-ban sentence with its shortest anchor, "never the way forward", which tells the non-interactive branch apart.
- `TestEngine_Fix_TableShapeAndFiles` in `fix_test.go`: drop the "`fix` is the exact change to make and where" check on `review-format.md`, and assert instead that the written format file is non-empty, so the test still fails when the format file is not written.
- The file comment heading `template_test.go`, which sits above `requireContains`, stops saying the assets' load-bearing statements are pinned in `prompt_test.go`: it says the Go-built blocks and the seat stencils' exclusion of the cluster block are pinned from full compose renders in `prompt_test.go`.

**ImpactSummary:** test-only, inside `internal/burlerengine`'s `prompt_test.go`, `fix_test.go` and `template_test.go`; no production code or helper signature changes.
