```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over `checkRef` and `rootRelativeGlyphRemedy` in `internal/planparser/validate.go`, case by case, without tools.

**Per function and branch: case run, verdict**

- `checkRef` (inlined `report`), `Uses` `guide.md#` under `root: docs`: `diskPathForRef` gives `("guide.md", true)`. `satisfied` is false, so one finding is appended with `Check` `"path-missing"`, `Card` `cardID(c)` and `Ref` `raw`. This is the same triple the old closure produced. Correct.
- `checkRef`, `Rename` `Old` and group `Refs`: the same function body is reached from every call site. Correct.
- `checkRef`, `Detail` format: the only test that matches detail text is at `validate_test.go:1622`, a substring on a file name, and the new text still contains it. Correct.
- `rootRelativeGlyphRemedy`, `root` empty or `"."`: returns `""`. A root of `"./"` cleans to `"."`, so it also returns `""`. Correct.
- `rootRelativeGlyphRemedy`, `raw` is a path-shaped slashed extensionless directory such as `docs/internal/foo`: `ParseRef` errors because there is no `#`, so it returns `""`. Correct.
- `rootRelativeGlyphRemedy`, `language: none`: `diskPathForRef` is already not-ok, so the remedy is never reached. `ParseRef` would also error. Correct.
- `rootRelativeGlyphRemedy`, `raw` is a handle, bare symbol or member glyph: `diskPathForRef` skips all three, so the remedy is never reached. Correct.
- `rootRelativeGlyphRemedy`, `raw` `guide.md#`, root `docs`, `docs/guide.md` present: `joined` is `docs/guide.md`, `satisfied` is true, and `glyph.Self(Go, joined)` gives `docs/guide.md#`. The remedy is correct and the suggested plain path `guide.md` is root-joined to the same file. Correct.
- `rootRelativeGlyphRemedy`, `joined` present only as a `Create` target, `Rename` `New` side or `Part.Paths` entry: these sets are keyed by root-joined normalized paths and `satisfied` consults all of them. Correct.
- `rootRelativeGlyphRemedy`, `..`, absolute or backslash root such as `../x`, `/abs` or `docs\sub`: `joined` can stat outside the worktree. Quarry's `glyph.Self` rejects dot segments, empty segments and bad runes, so the error path returns `""`. This relies on quarry's rejection, not on any check in the diff. Correct.
- `rootRelativeGlyphRemedy`, `csharp` plan with a self glyph: `ParseRef` returns the Go-alphabet self glyph, so `parsed.Lang` is `Go` and `Self` succeeds. Correct.
- `rootRelativeGlyphRemedy`, `joined` equal to `mapped`: cannot happen, because `root` is non-empty and not `"."`. Correct.
- `rootRelativeGlyphRemedy`, swallowed `ParseRef` and `Self` errors: the remedy is an optional hint, the original finding is still emitted, and the error is not a defect signal. Not a finding.
- `rootRelativeGlyphRemedy`, `//`-escaped path: see Finding 1.

**Findings**

1. **LOW, design.** `internal/planparser/validate.go:1356-1381`, `normalize.go:42-44,169-193`. The remedy text claims the writer spelled a glyph relative to `root:`, but a `//guide.md` path is not distinguishable from `guide.md#` after normalization and canonicalization.
   - Input: under `root: docs`, a card writes `//guide.md`, meaning the worktree-root file, which is missing. `docs/guide.md` exists.
   - Result: the ref reaches `checkRef` as `guide.md#` and the remedy fires. It says "a glyph is never joined to root: ... drop the trailing "#"", although the writer wrote no `#` and no glyph.
   - Effect: the hint repoints the card at a different file than the one the writer named, and it is worded as a fix for a glyph-spelling mistake. The verdict is unchanged and the advice is still a valid way to name `docs/guide.md`, so this is a wording-accuracy issue, not a wrong verdict.
   - The doc comment's claim "raw is no glyph" does not hold for this case. No existing test pins it.

No other defect found in this lens.

Tool calls: none, because the context already held everything the lens needed.
