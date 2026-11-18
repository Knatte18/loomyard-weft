---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: The subjectDiff error pass-through row has no target once F4 removed subjectDiff, so StageDiffGivesTheCommittedDiff gets a row whose CodeDiff errors and asserts an empty Diff instead; every other row is added as proposed.
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

## Changes

- F1: `//testtiming:keep` lines above `TestTemplate_FillsWithAllMarkers` and `TestTemplate_OptionalDirectives`.
- F2: `diffPieceMaxBytes` lowered to 40,000 and `diffPieceMaxLines` (1,500) added.
  A `pieceCap` carries both, and `boundedUnits` and `packUnits` close a unit or text on either.
  The constant's comment and the `doc.go` paragraph match.
  The summary the finding names is not in the worktree, so it is not updated.
  Rows: a package over the line cap, and many short lines over a line cap, both under a byte cap they pass.
- F3: the stencil opens with the task's full diff against its merge base and defers to the rubric's range where it narrows or refuses it.
- F4: `Fabric.ReadCommittedDiff` reads `git diff ... <base> HEAD`.
  `Env.CodeDiff` carries it, wired in `internal/loomcli/wiring.go` through `committedCodeDiff` over a new `recordedTaskBase` that `codeSubject` shares.
  `stage_diff` requires `Env.CodeDiff` in place of `Env.CodeSubject`, and `subjectDiff` is gone.
  The Bouncer's `CodeSubject` patch is unchanged.
  `envkit` lists `CodeDiff` as nil-legal and defaults it.
  Tests: `TestFabric_ReadCommittedDiff` (committed change in; uncommitted edit and untracked file out), and the `StageDiffGivesTheCommittedDiff` rows.
- F5: `FileOverTheCapSplitsAtHunksThenLineEnds` pins exact piece contents, with a hunk-first then line-end split.
  The assisted-round test sets a two-package diff and asserts the chair's and the assistant's explore steps name the same pieces.
- F6: `TestWriteDiffPieces` rows for a directory and a piece that cannot be written; `TestEngine_Review_StagesTheDiff` with a root file and a split package asserting `(the repository root)` and `part N of 2`;
  a `CodeDiff` error row; a producer subtest with a counting reader over a died-then-done runner asserting one read and the same `Diff` on both attempts.
- F7: a split file's header, the text before its first hunk, opens every piece after its first.
  It is folded into the file's first unit, so no piece ends on a bare header.
  A file with no hunk, or a header past half a piece, repeats nothing, and the docs say so.
  The stencil names the line cuts and the repeated header.
- F8: `diffFilePackage` reads the `+++` line, or the `---` line for a deletion, unquoting a C-quoted path, and falls back to the `diff --git` line for a section with neither.
  Row: `docs/foo b/bar.md` groups under `docs/foo b`.
- F9: `pieceFileName` keeps the last `diffPieceNameMaxBytes` (80) bytes of the flattened package.
  Row: a package path over 300 bytes is written.
- F10: `runeBoundaryAtOrBefore` steps an over-cap line's cut back to a rune start.
  Every piece in `TestWriteDiffPieces` is checked as valid UTF-8, with a multibyte-line row.
- F11: the doc of `TestTemplate_FillsWithAllMarkers` names no excluded marker.

Gate repair: comments in `diffpieces.go`, `diffpieces_test.go`, `burlerengine/doc.go`, `template_test.go`, `taskpatch.go` and `shedrecipe/doc.go`
were re-broken one sentence per line, with no line at 120 characters; no code changed.
A second gate pass rewrote the `taskpatch.go` file header as two shorter sentences.

Commits are grouped by file set, since F2, F5, F7, F8, F9 and F10 share `diffpieces.go` and its test.

## Self-check

- fix-consistency, LOW: the stencil names neither line cuts nor the repeated header. Fixed as proposed in the finding.
- fix-proposal, LOW: a piece could end on a bare header of the next file. Fixed by folding the header into the first unit, with a `SplitFileHeaderNeverEndsAPiece` row.
- fix-proposal, LOW: the docs claim a header on every continuation piece, wider than the code. Fixed by stating the half-piece condition in the `writeDiffPieces` comment and `doc.go`.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
