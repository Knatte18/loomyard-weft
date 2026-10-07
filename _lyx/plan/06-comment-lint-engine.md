# Card 6 — comment-lint-engine

**Create:**
- `internal/commentlint/doc.go`
- `internal/commentlint/lint.go`
- `internal/commentlint/diff.go`
- `internal/commentlint/lint_test.go`
- `internal/commentlint/lint_integration_test.go`
- `internal/commentlint/testmain_test.go`
- `internal/commentlint#Lint`

**Edit:**
- `docs/overview.md`

**Intent:**
Decision D7's engine: a Go package that finds fixed-column-wrapped line breaks in the `//` comment blocks a diff touches.
It is told the worktree and the base and resolves no cwd (`PATTERN-told-geometry`).

`Lint(worktree, base)` diffs `.go` files: with an empty `base`, the working tree against HEAD, untracked `.go` files included;
with a `base` commit, `base` against HEAD with rename detection, so a pure rename adds no line.
Git runs through `gitexec.Run` (`PATTERN-gitexec-checked-call`); the base and new text of each changed file are read through git, not the filesystem, for the base side.
A non-Go diff yields no findings.

A break between two consecutive lines of a `//` comment block is checked only when it is new: the words on its two sides are not adjacent across a line end in the base text.
A line end an edit carries over unchanged is not new, so fixing one flagged break never makes another break of the block checked, and a legacy wrapped block is flagged only where an edit creates a break.

A finding is a line at a checked break, followed by another prose line of the same block, that ends neither a sentence (`.`, `!`, `?` or `:`, each optionally followed by a closing `)`, quote or backtick), nor at a semicolon, nor at a comma whose next line starts with a coordinating conjunction.
Skipped: directive comments (`//go:`, `//lyx:`, `//testtiming:`, `//nolint`), indented code blocks, doc-comment list items and headings, and generated files, as `tools/godocreflow` skips them; the engine re-implements those checks, since `tools/` is not importable.
`Finding` names the file, the line number and the line's text.

Bound: the lint lets every comma-plus-conjunction break through, compound predicates included, because telling them from independent clauses needs a parser; those stay review findings.

Tests: `lint_test.go` (untagged) drives the break classification over base and new text pairs:
a fixed-column-wrapped new comment is flagged; an untouched wrapped line and an untouched wrapped block are not;
once one flagged break of a wrapped block is fixed, its other breaks stay unflagged;
sentence-end (with a closing `)`, quote or backtick after it), colon, semicolon and comma-plus-conjunction breaks pass;
directives, code blocks and list items are never flagged.
`lint_integration_test.go` drives `Lint` over a real repository: the worktree form sees an untracked file, the base form sees committed changes, and a pure rename flags nothing.
`testmain_test.go` runs the package through `tmuxkit.Main` under `gitkit.HermeticGitEnv()` (`PATTERN-test-isolation`).

`doc.go` carries the package design; `docs/overview.md`'s package tree lists `internal/commentlint`.

**ImpactSummary:** New package and one overview line; nothing calls it yet.
