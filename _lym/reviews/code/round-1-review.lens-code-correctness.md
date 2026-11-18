```yaml
lens: code-correctness
outcome: reported
```

code-correctness lens: I ran each function and branch the diff adds or changes against concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `writeDiffPieces`, empty or whitespace diff (`" \n"`): returns nil and creates no directory. Correct.
- `writeDiffPieces`, failure of `MkdirAll` or `WriteFile`: returns the wrapped error, and `Engine.Review` returns it. The pieces already written stay in the fresh instruction directory, but nothing reads them. Correct.
- `writeDiffPieces`, two packages interleaved (alpha, beta, alpha, root): one piece per package, in first-seen order, with seq-prefixed names. Correct.
- `pieceFileName`, packages `internal/a-b` and `internal-a/b`: both flatten to `internal-a-b`. The seq prefix (`001-`, `002-`) keeps the names unique. Correct.
- `diffFilePackage`, quoted path with a space (`"a/x y" "b/x y"`): quotes are stripped and the package is `.`. Correct.
- `diffFilePackage`, a new-side path that holds ` b/` (such as `docs/foo b/bar.md`): `LastIndex(" b/")` lands inside the path. The file gets the wrong package: it is labelled and grouped as `foo`'s sibling, or as root. Wrong, cosmetic (see F4).
- `diffFilePackage`, section with no `diff --git` header (leading text): goes to `rootPackage`. Correct.
- `boundedUnits`, file under the cap: one unit. Correct.
- `boundedUnits`, file over the cap: splits into the header chunk and then one chunk per hunk. A hunk over the cap splits at line ends. Every byte is kept. Correct as to bytes, but a continuation piece carries no file header (F2).
- `boundedUnits`, one line over the cap: cut at a byte offset of `maxBytes`. A multibyte UTF-8 rune can be split across two pieces (F3).
- `boundedUnits`, binary patch (`--binary`): its base85 lines start with letters, never `@@ `, so it splits at line ends. Correct as to bytes.
- `packUnits`, units each at or under the cap: greedy packing, and no text passes the cap. Correct.
- `splitBeforeLines`, text with no trailing newline: the last chunk is kept. A prefix on the first line makes no empty leading chunk. Correct.
- `formatDiffPieces`, root package and a multi-part package: labels and "part i of n" render. Correct.
- `diffPiecesBlock`, no pieces: returns `""` and reads nothing. A fill error propagates. Correct.
- `Engine.Review`, `p.Diff` empty: no directory, no marker, the same explore step as before. Correct.
- Assistant `composeAssist`: `assistProfile` copies `*p` after `p.diffPieces` is set, so the assistant's explore step names the chair's pieces. Correct.
- Probe and resume (`probeTable`): composes nothing, and the files already written are reused. Correct.
- `BurlerReviewProducer.readDiff`: a nil reader gives `""`. A read error is logged and gives `""`. It is called once per `Call`, after the hand-on and probe exits and before the attempt loop, so attempt 2 reuses it. Correct.
- `subjectDiff`: an error gives `("", err)`, and not found gives `("", nil)`. Found gives the patch. Correct.
- `burlerReviewEntry`: a non-bool `stage_diff` is refused by `configBool`. `stage_diff: true` with a nil `CodeSubject` is refused by `requireSeam`; a typed-nil func is caught through reflect. An absent key reads no diff. Correct.
- Range alignment: the pieces' base (`codeSubject` → `taskMergeBase`) is the same seam as the rubric's `review_base` (`verifyMergeBase = taskMergeBase(location)`, `wiring.go:776`/`913`). The patch is the working tree plus untracked files, not `base..HEAD`, so it differs only when the tree is dirty at review time (F5).

**Findings**

- **F1 — MEDIUM, design — `internal/burlerengine/diffpieces.go:141-144`.** A piece under `diffPieceMaxBytes` does not always read whole, which is the guarantee the change exists for.
  - The cap counts bytes only. An agent's Read also stops at a default 2,000 lines, and it caps tokens, not bytes.
  - A 60,000-byte piece of short lines (YAML, `go.sum`, lists, about 25 bytes a line) is about 2,400 lines, so a default Read truncates it.
  - A dense piece, such as a `--binary` base85 patch or hash-heavy lines at about 2 bytes per token, plus the line-number prefix per line, passes about 25,000 tokens and is refused.
  - That is the failure the board entry's Goal describes.
  - Evidence: `diffpieces.go:144` (`const diffPieceMaxBytes = 60_000`), `boundedUnits` and `packUnits` measure `len()` only (`diffpieces.go:243-282`), and `taskpatch.go:84` renders `--binary`.
  - Fix: cap lines as well as bytes, or lower the byte cap so the line and token limits hold too.
- **F2 — LOW, design — `internal/burlerengine/diffpieces.go:240-264`.** When one file is split, every piece after the first starts mid-file, with no `diff --git` or `+++` header.
  - Read alone, its hunks cannot be attributed to a file.
  - When the cut is at a line end inside a hunk, the piece also lacks the `@@` line numbers.
  - The test `FileOverTheCapSplitsAtHunksAndLines` checks only that the pieces join back (`diffpieces_test.go:392`).
- **F3 — NIT, design — `internal/burlerengine/diffpieces.go:254-256`.** `line[:maxBytes]` cuts at a byte offset, so a multibyte UTF-8 rune can be split across two pieces. Each piece then holds invalid UTF-8 at the cut.
- **F4 — NIT, design — `internal/burlerengine/diffpieces.go:230-238`.** `diffFilePackage` takes `LastIndex(" b/")` of the header line. A new-side path that itself holds ` b/` gets the wrong package and is grouped and labelled under it. The bytes still land in a piece.
- **F5 — LOW, design — `internal/shedrecipe/entries_burler.go:61-69`, through `fabricengine/taskpatch.go:84-99`.** The staged patch is the working tree against the base, plus non-ignored untracked files. The rubric's range is the committed `git diff <base>..HEAD`.
  - On a dirty tree at review time, the pieces hold uncommitted or untracked content that the range does not.
  - The stencil tells the reviewer to read the pieces in place of that `git diff` (`burler-diff-pieces.md:57`), so the reviewer would judge the wrong subject.

**Tool calls**

1. `grep review_base` in `internal`: to find where the rubric's base is computed, which my context lacked.
2. `grep ReviewBase` in `loomcli` wiring: to find the seam that fills it.
3. `grep verifyMergeBase` in `wiring.go`: to confirm it is `taskMergeBase`, the same base the code subject uses.
