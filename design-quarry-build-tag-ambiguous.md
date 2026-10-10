# Build-tagged member declared once per tag set resolves ambiguous, so its plan Create can never pass record-batch

`quarry-build-tag-ambiguous` · Note · bug

Imported from [issue #500](https://github.com/Knatte18/loomyard/issues/500).

Seen (test-suite-weight, 2026-10-10): plan card 10 declares `plan:internal/testkit/tmuxkit#tmuxAllowed` as a Create, implemented the idiomatic Go way: `const tmuxAllowed = true` in `allow_tmux.go` (`//go:build tmux || llm`) and `const tmuxAllowed = false` in `allow_none.go` (`//go:build !tmux && !llm`).
`lyx quarry resolve internal/testkit/tmuxkit#tmuxAllowed` returns `status: ambiguous` with both files as candidates, because quarry ignores build constraints, and record-batch's DoneChecks accept only `found` or `multipart`, so the batch fails `create-not-done` however it is implemented.
The recovery strand stopped to ask, and nobody in the run may edit `_lyx/plan` to drop the glyph.

Workaround: a parent ruling to restructure the code into one `var` plus an `init()` in the tagged file, listed as deviations.

Expected: a member declared once per mutually exclusive build-constraint set resolves (quarry honours build constraints, or DoneChecks accept an ambiguous answer whose candidates all sit in files with disjoint build tags); and the plan validator flags such a glyph at Plan time rather than at record-batch.
