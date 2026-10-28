```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **`stripHandles` / `strippedRefs`:** the old code rewrote `refs` in place, but `refs` was always a fresh slice built by `append` from nil, so it never aliased `group.Refs` or `card.Uses`. The new copy returns the same values. For `["plan:a#B", "c"]` it returns `["a#B", "c"]`, and an empty or nil input returns an empty slice.
- **`coversSeamTest`, glyph route:** this is unchanged and still iterates `strippedRefs(card)` over every group and Uses.
- **`coversSeamTest`, file route:** it now reads `strippedRefs(card, Delete, Rename, Move)` plus `stripHandles(card.Uses)`. With types given, `strippedRefs` skips Uses, so there is no double count. A card with Edit `internal/c/c_test.go` gets no file cover, which matches D6 and check 55. A card with Uses `internal/c/c_test.go` is still covered (row `TestFileUsesPasses`). A Delete, Rename or Move naming the file is still covered.
- **Callers:** the signatures of `strippedRefs` and `coversSeamTest` are unchanged. `touchesStruct` and the later-cards loop use `strippedRefs` with the same arguments and results as before.
- **New row `SeamTestEditingTheTestFileFails`:** with the old file route in place, the `Edit internal/c/c_test.go` ref matches `seam.File`. `coversSeamTest` then returns true, no finding is emitted, and the row's `wantFiles` expectation fails. So the row fails with the defect in place. It passed after the fix in the gate run (tier0 324 passed).
- **Out of scope:** no row covers a Rename or Move of the test file. The review did not ask for one.

Tool calls made: none; my context already held every changed file and the gate output.
