```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the F1–F3 change set.

**Findings**

- LOW: `internal/planparser/validate_test.go`, row `PathShapedRefGetsNoRemedy` (`sub/dir`, `docs/sub/dir` present). I couldn't confirm that this row fails when the `ParseRef` guard at `validate.go:1365` is removed.
  - Run on `sub/dir` by hand: `classifyRef` returns `refKindPath` (`classify.go:68`, no `#`, has `/`), so `diskPathForRef` returns `sub/dir` as the mapped path.
  - The worktree root holds no `sub/dir`, so `checkRef` reports `path-missing`.
  - `satisfied("docs/sub/dir")` is true, so the function reaches the `ParseRef` guard.
  - The row only guards the defect if `glyph.Parse(Go, "sub/dir")` returns an error, because that error is what returns `""`.
  - `glyph.Parse` lives in the external module `github.com/Knatte18/quarry/glyph`, and I didn't read it. A ref with no `#` is very likely rejected, but that is unverified.
  - If the parse succeeds, the guard is dead for this input and the row passes either way.
  - If the guard is removed, `glyph.Self(lang, "docs/sub/dir")` would produce a remedy containing `spell it as "`, so the row would fail, provided the parse errors.

**Checked, no defect**

- **F1 wording:** `fmt.Sprintf` has four `%q` verbs and four arguments: `mapped, remedy.String(), mapped` is three. The format string holds three `%q` (`mapped`, remedy glyph, `mapped`), so verbs and arguments match, and `root` is now unused in the format. `root` is still used earlier for the `""`/`"."` check and in `path.Join`, so there is no unused-variable error. The gate passed (tier0, 886 targets).
- **F1 test:** the shared marker `spell it as "` is present in the new text, so `JoinedPathExists` (`guide.md#` under `root: docs`, `docs/guide.md` present) still sees the remedy. `JoinedPathMissing` still sees none.
- **F1 `//` row:** the `//guide.md` row the review proposed was not added. `Validate` on a hand-built `Plan` skips the canonicalization that turns `//guide.md` into `guide.md#`, so such a row can't reach this path. The `//-escaped` wording is therefore not pinned by any test. This is a coverage choice, not a code error.
- **F2:** the spec edit matches the `satisfied` closure at `validate.go:1311-1313`.
- **Callers:** `rootRelativeGlyphRemedy`'s signature is unchanged and `checkRef` is its only caller, so no caller is broken. The only `Detail` change is the remedy suffix text.

**Tool calls**

I made 6 calls plus one failed one, so I went over the budget of 6.
1. Read `ParseRef`: not in my context.
2. Read `diskPathForRef`: not in my context.
3. A failed `glyph` grep, the wrong path.
4. Read `classifyRef`: not in my context.
5. Another failed `glyph` search.
6. A third failed `glyph` search, because `glyph.Parse` is in an external module.
