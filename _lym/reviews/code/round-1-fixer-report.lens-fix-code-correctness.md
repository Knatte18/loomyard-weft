```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code through the correctness lens, running each changed function or branch on a concrete input.

**LOW** — `internal/testcover/blockkey_test.go`, two-file block in `TestKeyBlocks` (F6's cross-file ordinal assertion)
- The assertion `crossFile[0].Ordinal != 0 || crossFile[1].Ordinal != 0` cannot fail from the defect it is meant to catch.
- `BlockKey` includes `File`, and `KeyBlocks` counts ordinals in `seen := map[BlockKey]int{}`. A p.go key and a q.go key are therefore distinct map keys even if the `byFile` grouping were removed.
- Cross-file counting would still give both keys ordinal 0.
- The test's `File` assertion is the part that bites, and only for a defect in `name := path.Base(b.File)`.
- The ordinal-per-file claim in the review and the test comment is not pinned by this case. The test is not wrong, only weaker than described.

The rest of the change set holds.
- **F2, `spliceReport`**
  - `renderReport(tags, nil)` returns the header text ending in one `\n`, the same shape `splitReport` produced.
  - The tag check still runs on `existing` first.
  - Stale-header run: the stale header is replaced and sections `a` and `b` follow. The result equals `renderReport(tags, []pkgReport{a, b})`.
  - The test fails with the old code, because the extra "Judged over the whole module." line would survive.
  - The `TestSpliceReport` rows are unaffected, since their existing report already carries the current header.
- **F3, second `WriteMap`**
  - `BuildFailure` is non-empty, so it is serialized and round-trips.
  - The fewer `Tests` and `Blocks` mean a merge-instead-of-replace defect would fail the `DeepEqual`.
  - The planted `newer_key` is ignored on read, as before.
- **F4, `judgePackage` second case**
  - `TestSmall` covers p1, and `TestBig` covers p1, p2 and q1.
  - Package view: `TestSmall`'s only block is also `TestBig`'s, so `TestSmall` is a candidate and removable, with covering `[TestBig]`.
  - Module view: `TestSmall`'s only block, p1, is also in `TestBig`, so `moduleContained` is true.
  - `TestBig` covers p2 alone in its package, so it is not a candidate. The earlier failure with `TestBig` covering only p1 is fixed, and the gate passes.
  - The keep text is carried and does not alter the classification.
- **F5, skipped test with profile**
  - The stand-in writes the profile and prints `--- SKIP ... (0.00s)`.
  - `RunAlone` sets `Action` to skip and `Elapsed` to 0, opens the profile and parses it. The result matches the expected row.
  - An early return for skips would leave `Profile` nil and fail the row.
- **F6, generic receiver**
  - The `{` of `func (t T[K]) Do() int {` is at column 24 on line 7, as the test uses.
  - Without the `IndexExpr` case, `receiverTypeName` would fall to `types.ExprString` and give `T[K].Do`, so the `T.Do` assertion fails with the defect in place.
- **F1:** the doc-only edit has no code reach.

Further tool calls: none. The context already held every changed file.
