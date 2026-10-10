# Build-tagged member declared once per tag set resolves ambiguous, so its plan Create can never pass record-batch

`quarry-build-tag-ambiguous` · Note · bug, webster, loom, quarry

Imported from [issue #500](https://github.com/Knatte18/loomyard/issues/500).

Seen (test-suite-weight, 2026-10-10): plan card 10 declares `plan:internal/testkit/tmuxkit#tmuxAllowed` as a Create, implemented the idiomatic Go way: `const tmuxAllowed = true` in `allow_tmux.go` (`//go:build tmux || llm`) and `const tmuxAllowed = false` in `allow_none.go` (`//go:build !tmux && !llm`).
`lyx quarry resolve internal/testkit/tmuxkit#tmuxAllowed` returns `status: ambiguous` with both files as candidates, because quarry ignores build constraints, and record-batch's DoneChecks accept only `found` or `multipart`, so the batch fails `create-not-done` however it is implemented.
The recovery strand stopped to ask, and nobody in the run may edit `_lyx/plan` to drop the glyph.

Workaround: a parent ruling to restructure the code into one `var` plus an `init()` in the tagged file, listed as deviations.

Expected: a member declared once per mutually exclusive build-constraint set resolves (quarry honours build constraints, or DoneChecks accept an ambiguous answer whose candidates all sit in files with disjoint build tags); and the plan validator flags such a glyph at Plan time rather than at record-batch.


## From issue #507

Imported from [issue #507](https://github.com/Knatte18/loomyard/issues/507).

A Go symbol with one body per GOOS (for example `StartTime` in `internal/proc`, or `runChild` and `terminatingSignals` in `internal/gatecli`) is answered `ambiguous` by `lyx quarry resolve`.
A plan can therefore neither `Edit` nor `Delete` such a symbol by glyph.
The only spelling the gate accepts is the file path, so retiring the symbol means deleting whole files.

The gate then refused `create-already-exists` for a `plan:` handle whose name matched a symbol those deleted files declared, because the resolve pass reads the tree before any card runs.
The plan had to rename the replacement symbols and edit their callers, a rename the design never asked for.

This is abnormal: the same card both deletes every declaration of the name and creates it again, and the plan is coherent.

What to do, either of:
- let a `Create` handle reuse a name whose every declaration the same card deletes;
- make `lyx quarry resolve` treat per-GOOS bodies of one symbol as a single declaration, so glyph `Edit` and `Delete` work.

Provenance: task `driver-context`; note `Plan-Write.md`.
