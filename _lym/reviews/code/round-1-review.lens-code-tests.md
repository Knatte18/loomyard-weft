```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test row that would fail if it broke, or "none". I made no tool calls.

**Per behaviour: test that pins it**

1. Remedy appended when `root:` is not `.`, `raw` is a self glyph and the `root:`-joined path is present.
   Pinned by the `JoinedPathExists` row of the new subtest, `internal/planparser/validate_test.go:1525` and `:1544`. It asserts the substring `spell it worktree-relative as "docs/guide.md#"`.
2. No remedy when the joined path is not satisfied.
   Pinned by the `JoinedPathMissing` row, `validate_test.go:1526`.
3. Remedy absent when `root:` is absent or `.`.
   No test row, but breaking it changes nothing. With root `.`, `path.Join` returns `mapped`, which is already unsatisfied, so `satisfied` returns false and the function returns `""` anyway. Not a gap.
4. `ParseRef` error gives `""`, so a path-shaped ref (not a glyph) never gets the remedy.
   **None.** The only `root:` test uses a glyph ref (`guide.md#`). If the `ParseRef` gate were deleted, a path ref like `x` under `root: docs` would pass through. `diskPathForRef` maps it to `docs/x`, and `path.Join("docs", "docs/x")` could be satisfied. The function would then emit a remedy for a non-glyph, and no row fails.
5. `glyph.Self` error gives `""` (`validate.go:1373`).
   None. It is nearly unreachable, because the unit already parsed. Not worth a row.
6. Non-Go alphabets: under `csharp`, `parsed.Lang` comes from the Go fallback in `ParseRef`, and under `language: none` there is no finding.
   None. No `root:` row runs under `csharp`, and none under `none` asserts the remedy is absent. The `csharp` subtest at `validate_test.go:1507` has no `root:`.
7. The wording past the first clause is not asserted.
   The `root: %q` clause and the "or drop the trailing `#` so root: joins the path `guide.md`" clause (`validate.go:1378`) are not asserted by any row. The test asserts only the first clause, so a wrong `mapped` or `root` in that text would pass.
8. The joined path counts as present via a `Create` target or `Rename` destination, not only on disk.
   The decision record claims this (`decision-record.md`, "What changes"), and the code gets it from the shared `satisfied` closure. Only the on-disk case has a row (`materializeFiles`). The union paths are exercised for the verdict elsewhere, but not for the remedy.
9. The inlined `report` closure (`checkRef` now builds the `ValidationError`) must keep the same `Check`, `Card`, `Ref` and `Detail` prefix at every call site (`Uses`, group refs, `Rename` `Old`).
   Pinned only by count: `validate_test.go:1500`, `:1569` (`Old` side) and `:1590`. `Detail` is checked only for a file name at `:1622`. No row asserts `Card` or `Ref`, but that is unchanged behaviour, not new. No consumer matches the old `Detail` string. The new subtest runs only through a `Prosa` group, so the remedy's reach to `Uses` and `Rename` `Old` is not pinned. It goes through the same closure, so this is minor.
10. A `//`-escaped path (`//guide.md` under `root: docs`) gets the remedy.
    None. Normalization canonicalizes `//guide.md` to `guide.md#`, so the remedy is emitted when `docs/guide.md` exists. It then tells the writer a glyph is never `root:`-joined, which does not describe the card as written. No row pins or rules out this case.

**Duplicate coverage:** the new subtest repeats nothing in `TestValidate_PathMissing`. The existing subtests count findings and never exercise `root:`.

**Findings**

- LOW, `design`, `internal/planparser/validate_test.go:1517`: no row covers a path-shaped (non-glyph) ref under `root:` with the joined path present. That row is the only one that fails if the `ParseRef` gate in `rootRelativeGlyphRemedy` (`validate.go:1365`) is removed. Evidence: the table at `validate_test.go:1520` has glyph rows only, and `validate.go:1365`.
- LOW, `design`, `internal/planparser/validate_test.go:1544`: the assertion checks only the first remedy clause. The `root:` quote and the plain-path alternative at `validate.go:1378` are unpinned. Evidence: `validate_test.go:1519` (`remedy` string) and `validate.go:1377`.
- NIT, `design`, `internal/planparser/validate_test.go:1525`: no row for a joined path satisfied through a `Create` target or `Rename` destination (behaviour 8). Evidence: `validate_test.go:1525`, `validate.go:1311`.

Tool calls: none. The manifest already held the diff, `validate.go`, `validate_test.go` and `shape.go`.
