# Round 2 fixer-report

## Changes

- F1 (LOW, scope): fixed as proposed.
  Added the `LinkedTestFileEmbedIsNoSource` row to `TestSelectorSelect` in `internal/testcover/testselect_test.go`, after `EmbedPackageDoesNotParse`.
  The row replaces `a/assets.go` with `package a\n` and `a/a_test.go` with a file holding `//go:embed static/page.txt`, changes `a/static/page.txt`, and expects no miss and no tests.
  It fails if `embedPatterns` reads a linked package's test files.
  Gate: `lym gate test ./internal/testcover` passed.
  Commit: `testcover: fix F1 — pin that a linked package's test files are not embed sources`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
