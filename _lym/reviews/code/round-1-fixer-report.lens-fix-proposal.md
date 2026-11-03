```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F1, F2 and F3 as the fixer carried them out.

**Findings**

1. **LOW**, F1: the `fix` key's `//guide.md` row was not added, and nothing marks that.
   - Changed text: the F1 `fix` key in the review (a `//guide.md` row under `root: docs`, asserting the new wording), against the subtest rows at `validate_test.go:1517`.
   - The fixer added a `PathShapedRefGetsNoRemedy` row for F3. For F1 it only changed the wording and the `remedy` marker. The `//`-escaped scenario that motivated F1 has no row.
   - Evidence: the new rows are `guide.md#` and `sub/dir`. The marker `spell it as "` is also met by the `//`-free `guide.md#` row, so no assertion tells the `//` wording from the glyph wording. If the `//` row cannot be built through `Validate` on a hand-built `Plan` (canonicalization runs in the parser, not in `Validate`), that is a "fixed differently" disposition. The fixer-report must name it, with that reason.

2. **LOW**, F3: the new row may not exercise the guard it claims to pin.
   - Changed text: row `PathShapedRefGetsNoRemedy` (`"sub/dir"`, `present: docs/sub/dir`, `wantRemedy: false`).
   - Evidence: the row only proves the `ParseRef` early return at `validate.go:1365` if `glyph.Parse(Go, "sub/dir")` returns an error. My grep did not find `glyph.Parse`'s source, so I could not confirm that. If it parses a `#`-less path as a symbol or path ref without error, the guard never fires on this row. The row then passes because the joined path is not satisfied, or `glyph.Self` fails. It would also pass with the guard removed.
   - Check before accepting: remove the guard locally and confirm the row fails. The review asked for exactly that property ("fails when the ParseRef guard ... is removed").

3. **LOW**, F1: the reworded remedy tells a plain glyph writer to drop an escape they never wrote.
   - Changed text: `rootRelativeGlyphRemedy` at `validate.go:1377`, "or drop the // escape or the trailing \"#\" so root: joins the path %q".
   - Evidence: for the common case of a writer who spelled `guide.md#` under `root: docs`, the `//` clause is noise. The sentence is now correct for both spellings, but it is longer than either case needs. It is a wording cost only, not a defect.

F2: no finding. The spec's "present on disk or created by a card" matches the `satisfied` closure at `validate.go:1311-1313`. That closure also includes a part path, which the spec phrase omits. "Created by a card" covers `Create` and `Rename` destinations but not a part path. That is a minor gap, LOW at most, and not worth a change.

**Tool calls**
- `grep -n "func ParseRef" -A40 internal/planparser/*.go`: read `ParseRef`, because my context did not show how the guard errors. It delegates to `glyph.Parse`.
- `grep -rn "func Parse(" ... internal/glyph/*.go`: failed, the path does not exist.
- `grep -rn "^func Parse(" ...` over the repo: matched only `partsparser` and `sleepjournal`. I could not locate `glyph.Parse`, so finding 2 stays unverified.
