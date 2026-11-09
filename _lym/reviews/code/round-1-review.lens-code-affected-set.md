```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines they touch.

**Per-change list**

1. **Stored answer value now leads with the escaped `$A` marker (template in `runButtonVia`).**
   - Set found:
     - the only reader, `buttonAnswer` in `buttonclick.go:264-269`, which now decodes;
     - the template-shape assertion in `buttonclick_test.go:57`, which is updated;
     - the answered-call rows in `buttonclick_test.go`, which now carry the marker;
     - the integration test's `stale` seed and check on the Escape row (`buttonclick_integration_test.go:162,189`), which doesn't read the decoded value and is unaffected;
     - docs: `doc.go:133-136` is updated; `doc.go:146,169,786` and `bar.go:21` only name the option as decision-bearing or unread, and stay true.
   - Covered.
2. **New `decodeStoredAnswer`, `buttonAnswerMarker` and `keptDollarEscape`.**
   - No other caller, reader or assertion exists.
   - A repo-wide grep for `lym_button_answer` and `%%%` found no stencil, spec or PATTERN background that describes the answer storage.
   - `pattern/PATTERN-reed-panes.md` has no match for `answer` or `%%%`.
   - The other hits are `shuttlefake.go` (a hit on `%%%`, not on the option) and research `.tsv` measurement files, which are not subjects.
   - Covered.
3. **`buttonAnswer` doc comment now says "decoded".** Updated in the same diff. Covered.
4. **Repository rules.**
   - `PATTERN-code-conventions`: the marker is one named constant used in both the template and the decoder.
   - No declared PATTERN changes were in the directive, so none are owed.
   - No stencil was changed.
   - Commit messages carry no `Pattern-Departure:` trailer and need none.
   - Covered.

**Findings:** none.

**Tool calls**
1. One `grep` for `lym_button_answer` and `%%%` across all files, plus a `grep` of `pattern/PATTERN-reed-panes.md` for `answer` and `%%%`. My context held only the `.go` and `.md` grep hits, so this added non-Go files (stencils, specs) and the PATTERN background.
