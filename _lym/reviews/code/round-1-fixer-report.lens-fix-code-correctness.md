```yaml
lens: fix-code-correctness
outcome: reported
```

no findings

I traced these paths by hand against the changed code, and none produced a wrong result:
- **`namesProject` / `isSameProjectFile`:**
  - Bare `a.csproj` matches by base name.
  - `./A/Tests.csproj` cleans to `A/Tests.csproj` and equals the project path.
  - A command word that is a path suffix of the project path matches, so a tier `dir` that differs from the project's base still works.
  - `B/Tests.csproj` against `a/tests.csproj` is false.
  - `Fast/bin/Debug/Fast.dll` sets `namesAny` without matching, so only a command with no path-like words still reaches every project.
  - Quoted and backslash words are normalised the same way as before.
  - The solution case still returns early, since it is checked first.
- **Callers:** `runReachesProject` and `lowestTierReaching` call `namesProject` with unchanged arguments. Run targets such as `./...` carry no extension, so they still reach every project.
- **`printRunWholeNotices`:** It now runs before both successful returns of `planTierRuns`, and the guard, vet and `--guards` refusals return before it. The text and the position relative to the other stderr lines are unchanged, so the `wantStderr` rows and the integration expectation that the notice precedes the envelope still hold.
- **F1 refusal text:** The new `fmt.Errorf` arguments match its format verbs, and the `exact` row in `tiers_test.go` equals the produced string. `roleClause` starts with `"; a session"`, so the join is well formed.
- **Added rows in `TestUnrunProjectLines`:** Each of these fails with the defect in place:
  - The `.dll` row fails because the old code returned true for it and gave no line.
  - The same-file-name row fails because the old base-name match found it reached.
  - The duplicate-selection row fails without `slices.Compact`, which would give two lines.
  - The empty-invocation row, the leading `./` row and the backslash row pin existing or new behaviour with correct expectations. The `./` row would also pass on the old code, so it guards only against regression.
- **New integration subtest:** The `runTest` wiring check fails if the `cfg.Scoper == "dotnet"` loop in `run.go` is dropped, because the expected line would be missing. Its expected way forward (tier1 takes targets) matches `lowestTierReaching`, and the package gate passed at tier0 and tier1.

Tool calls made: none; the context already held every file in its changed form.
