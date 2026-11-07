```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens on the new test rows in `TestEnsure_HubWideWalkRendersAndCommitsTheHeddleOverview`.

**LOW — row "overview already current" does not fail against the mutant it targets.**
- The changed text is the row's `wantOverviewCommitted: false`, which asserts that `log -1 --name-only` omits `HEDDLE.md`.
- The review says "The staging layer drops an unchanged path anyway, so an always-append mutant would pass."
- With `written = append(written, heddleengine.OverviewFileName)` made unconditional at `walk.go:264`, the bytes on disk are unchanged. The commit would still not name `HEDDLE.md`, so the row passes with the defect in place.
- The row does catch a render that rewrites identical bytes into a different state, or a stale overview that is not re-rendered.
- It cannot pin the `changed == false` branch itself.
- The reviewer's proposed `fix` carried the same limit, so this is a note and not a regression.
- Pinning the branch would take a seam or a log assertion, which the review did not ask for.

No other findings.
- **Rows "missing overview" and "stale overview":** these run unchanged logic, with the new flags set to the old behaviour (`failedWriteFirst`, `wantOverviewCommitted: true`).
- **Row "heddle store that does not load":**
  - Walked with `{not json` in `heddle.json`: `RenderOverview` returns an error, `renderHeddleOverview` returns false, and `Ensure` returns nil.
  - A regression that returned the error would fail `Ensure` and so fail the row.
  - The row also checks that `HEDDLE.md` is unchanged and that `landing` is committed.
  - Its assertion that `RenderOverview` still errors guards the fixture itself.
- **Gate:** the tier-1 run exited 0, and its output shows all four subtests ran.
- **Callers:** the change touches only the test, so no signature or caller is affected.

Tool calls: none. My context already held every changed file and the gate output.
