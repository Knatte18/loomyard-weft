```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, covering which test pins each behaviour the diff adds or changes, and any test-economy problem.

**Per behaviour: the test that pins it**
- **A card id ending in the fabric suffix is refused with a rename way forward.** Pinned by `TestCheckCardFabricReference`, row "card slug ending in the fabric suffix fails" (`fabricref_test.go:79`).
  - It asserts exactly one finding, check `card-fabric-reference`, and a Detail holding `card 1-prime<suffix>` and `rename the card`.
  - Breaking the new `MatchPath(id)` branch drops the finding, and the row fails.
  - Breaking the way-forward text fails the `rename the card` substring.
- **A slug that only holds the suffix mid-name passes.** Pinned by row "card slug holding the fabric suffix mid-name passes" (`fabricref_test.go:80`).
  - It asserts no findings for `prime<suffix>-marker`.
  - A matcher that matched mid-name would fail it.
  - This is the acceptance-criterion negative case.
- **Existing rows keep their outcomes.** Pinned by the 13 untouched rows in the same table (`fabricref_test.go:66-78`).
  - Each uses slug `alpha`, whose id never matches, so each still expects its old finding count.
  - A changed order or count would fail the `len(got) != 1` check.
- **The suffix is derived, not hard-coded.** `fabricSuffix` comes from `filepath.Base(fabricengine.RecordsWorktree(...))` with the `task` prefix trimmed (`fabricref_test.go:25`). A suffix change cannot silently invalidate the rows.
- **The finding's `Card` field is the card id.** No test pins it. The existing rows don't assert `Card` either, so this is not a gap the diff opened.
- **Callers that consume these findings.** `internal/planglyph/index.go:50,59` and the `internal/loomcli/parity_test.go` row only convert and forward the findings. The new finding adds no behaviour there that needs its own test.

**Test economy**
- **Duplicate coverage:** none. Neither new row repeats an existing row, since none of the existing rows varies the slug.
- **Per-helper tests:** none added. Both rows go through the exported `CheckCardFabricReference`.
- **Review-fix test:** none.
- **Stencil prose assertion:** none.
- **Uncovered new behaviour:** none.

**Findings:** none.

**Tool calls**
1. Read `internal/planparser/fabricref_test.go` lines 25-125, because my context held only the diff hunks and not the table's assertion loop or the existing rows.
