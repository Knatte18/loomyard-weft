```yaml
lens: fix-code-correctness
outcome: reported
```

**Lens fix-code-correctness: LOW on the F4 other-directory row, no other findings.**

- **LOW, `internal/planglyph/index_test.go`, row "format over a test file whose base name only another directory holds".**
  - **Summary:** the row probably does not exercise the `path.Base` match that F4 asked it to pin.
  - **Evidence:** `glyphlessFile` calls `TOC(worktreeRoot, "internal/x/x_test.go")`. With that file absent and only `internal/y/x_test.go` present, quarry most likely answers with an error or an empty `Files`. The row then passes through the toc-error or absent-file return, the same path as the "repository lacks the file" row.
  - **Missing case:** a toc answer whose `Files` holds a same-base entry from another directory is not reached, and that case is hard to build from a fixture. This is a coverage-strength gap, not a wrong result.
- **F1 clause removal:** the F1 rows would fail with the kept-range clause removed, so no finding. For insert second (position 2), the Spans row (cards 1–3) fails: `mapsFaithfully` would return true, card 3 would become 4, and the line would read "1–4" with no mention. The Pair row (cards 1–2) is only kept by `previous < e.position && e.position <= number`, so it would fail the same way. The insert-third row pins the boundary: Pair stays unlisted and unchanged.
- **F2 rows:** the row that faults `01-alpha.md` fails if the bare-only change is never written, because the fault is never hit and the call returns no error. The overview-fault row is the one that pins journaling. `renumber.go:360` writes in-place cards before the overview write at line 369, so a card changed only by a bare reference must already be journaled for the tree to be restored byte for byte. Both rows are sound.
- **F3 Card Index line:** delta's Summary holds `card <f+2>`, a card that moves in every insert and remove row. The existing Card Index summary comparison fails if the scan rewrote that line. Delta's `Intent` is set explicitly, so the card body is unaffected, and `TestRenumber_Refusals` and `TestRestoreRenumber` only compare trees before and after.
- **Callers:** `removeSecondMentions` and the `wantMentions` fields are used only by `TestRenumber`. The new `prepare` field is optional, so the other rows are unaffected. The gate reported tier0 as 1395 passed and 0 failed.

Tool calls made:
1. `grep` of write order in `internal/planparser/renumber.go`: my context lacked the order of the in-place card writes against the overview write.
