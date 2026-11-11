```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set. Findings checked: F1 and F3. F2's premise is a single ordering rule, not a set, so I left it unchecked.

- **MEDIUM — F3, `internal/gatecli/testmap_integration_test.go`, test `ACallEndedMidRunOrMidBuildRecordsNoEntryAndNoFailure`: the fix covers one of the three build-side cancel branches.**
  - The finding's premise is that removing any of the three checks in `mapPackage` would write a `BuildFailure` map for a call the user stopped, and the test would still pass.
  - The set is the three `r.ctx.Err()` checks at `testmappackage.go:110` (after `LinkedPackages`), `:135` (after `BuildBinary`) and `:143` (inside `buildLym`).
  - The new `WhileTheCoverageBuildHangs` row hangs the coverage build through `testMapHangBuildEnv`, so it reaches only `:135`.
  - No row hangs `go list`, which would reach `:110`, or the `lym` build, which would reach `:143`. Removing either check still passes every row.
  - The reviewer's `fix` proposed two rows, so the fixer carried it out as written. The premise still names three branches, and `testMapFailListEnv` and `testMapFailLymBuildEnv` show the stand-in already has switches for those two commands but no hang switch.
  - Whether to add the two rows or accept the gap is the fixer's call.
- **F1: no findings.** A search for the old sentence and its paraphrases (`counts from its map`, `from its map's build`, `no (recorded )?measurement time`) over `.go` and `.md` files outside `_lym` found only the two corrected statements, in `testmapaudit.go:18` and `doc.go:1112`. The `MeasuredAt` struct comment at `coveragemap.go:87` makes no claim about it.

Tool calls (2):
1. `grep` for the F1 phrasing across `.go` and `.md` files. My context held only the `doc.go` and `testmapaudit.go` text, not the rest of the repo.
2. `grep` for `ctx.Err()` in `internal/gatecli/testmap*.go`, for the F3 set of cancel checks. My context held `testmappackage.go` but not the other files' checks.
