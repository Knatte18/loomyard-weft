```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set search behind each of F1, F3, F4, F5, F6, F7 and F8 and found one gap.

**Findings**

- **LOW** — F1's `TapestryRetiredRows` entry `Loom-Preflight`, in `internal/reportengine/timingspans.go` (phase table around lines 87 and 124).
  - A past run's history or status that carries `Loom-Preflight` is still read by the report engine as `PhaseOther`.
  - The table maps only `Tapestry-Preflight` to `PhaseSetup`. Its retired-row handling loops over `DarnRetiredRows()` only, and `Merge-In` is covered separately by the literal `retiredMergeInRow`.
  - The fixer's entry fixes resume, `peers` and `arm` (`internal/loomcli/arm.go:197` and `internal/loomcli/peers.go:88`). It does not reach this reader, which has no `Loom-Preflight` literal and does not consult `TapestryRetiredRows`.
  - Whether to fix it is a judgment call. The review did not name this site, and the missing mapping may predate this round (commit `edea6b48`).

**Sets that came back covered**

- **F7:** the search found no remaining recipe-valued `loom` in Go test or non-test code.
  - The remaining hits are config-module keys and are correctly kept: `configsync_test.go` and `loomengine/recipeoverride_test.go` map `"loom"` to a module document, and `escalation_test.go` tests `module == "loom"`.
  - The one hit that might be a recipe value is `internal/boardengine/legacy_test.go:256`, which feeds `"type":"loom"` through `decodeLegacy` as a legacy record, so it is legacy data, not a recipe name. It is the same shape as the fixture the fixer changed in `board_test.go:184`, so it is a judgment call.
- **F3 and F4:** no default-recipe or seed `loom` is left in `README.md`, `docs/overview.md`, `docs/*.md` or `tools/sandbox`.
  - The remaining `docs/overview.md` hits (lines 626 and 755) describe the general run or the config module, not the recipe value.
- **F5, F6 and F8:** the targeted lines are the only sites of their kind in the searches.

**Calls made**

1. A grep over Go and test files for `loom` used as a recipe value or a recipe-keyed map key. I needed it to check F7's set beyond the files the review named.
2. A grep over `README.md`, `docs/`, `tools/sandbox` and the recipe spec for a default or seed `loom`. I needed it to check F3, F4 and F6.
3. A grep for the `Merge-In`, `Loom-Preflight` and `RetiredRows` consumers across `internal`, `contracts`, the docs and `README.md`. I needed it to check F1's callers.
4. A read of `internal/reportengine/timingspans.go` and a grep of the `NameTapestryPreflight` readers. My context did not hold them, and I used them to confirm the gap in the F1 finding above.
