```yaml
lens: code-correctness
outcome: reported
```

**Task:** check the correctness of the code the diff adds, one concrete input per branch (lens code-correctness).

**Per-branch list**

- `JudgeMap`, empty map (`Tests` nil): `out` is empty, `Judge(nil)` returns empty, so the result is empty. Correct.
- `JudgeMap`, keep-marked entry: reason is `ReasonKept`, and the entry is not in `runs`, so it counts as neither judged nor cover. Correct.
- `JudgeMap`, a name in `keeps` that no entry carries: ignored. Correct.
- `JudgeMap`, entry with `Outcome` set: reason is the outcome, not judged, not a cover. Correct.
- `JudgeMap`, `OutOfProcess` or `Unresolved` spawn entry with no outcome: goes through `Judge`, gets `notJudgedReason`, and is excluded from the counts. Correct.
- `JudgeMap`, passing entry with no blocks: `ReasonNoBlocks`. Correct.
- `JudgeMap`, block index out of range: would panic, but `runRedundant` runs `checkMapBlocks` first, which covers negative and too-large indexes. Correct.
- `JudgeMap`, index alignment: `Judge` returns `len(runs)` verdicts and `at[j]` maps them back to map order. Correct.
- `runRedundant`, `lymcwd.CwdFrom` error: a refusal naming a way forward, exit 1. Correct.
- `runRedundant`, `resolveHub` error and the not-in-hub case: refusals with a way forward. Correct.
- `runRedundant`, `LoadConfig` error: a refusal. Correct.
- `runRedundant`, `redundancyModule` error: a refusal. Mostly correct. See F2.
- `runRedundant`, `pkg` `./cov` becomes `cov`. Correct.
- `runRedundant`, `pkg` `""` becomes `.` (`path.Clean`), reads the module-root map, and refuses with "no coverage map" if none exists. Correct, and consistent with `goModule.importPath`.
- `runRedundant`, `ReadMap` returns an error: `found` is false, `checkMapBlocks` is skipped, and the first `switch` case refuses as unreadable. Correct.
- `runRedundant`, map not found: refuses. Correct.
- `runRedundant`, map holds `BuildFailure` and no tests: refuses with the cause. Correct.
- `runRedundant`, `ScanKeeps` error: refuses with the file and line from the error. Correct.
- `runRedundant`, package directory absent from this worktree: `Glob` returns nothing, so keeps are empty and the verb judges without them. Correct and documented, since the keeps come from this worktree.
- `runRedundant`, verdict loop: `JudgeMap` returns one verdict per entry, so `m.Tests[i]` aligns. Kept, not-judged and candidate each go to their own list. Slices start non-nil, so empty lists print as `[]`. Correct.
- `runRedundant`, zero `BuiltAt`: formats as a zero RFC3339 time. Correct.
- `runRedundant`, `pkg` `../x`: `MapPath` joins it unchecked and escapes the maps directory. See F3.

**Findings**

- **F1 — LOW, design.** `internal/gatecli/redundant.go:99-107`: a map with `Failure != nil` (a later build failed and the old entries were kept) is judged like a current one, and the verb never says so. The answer shows `map_commit`, but nothing marks that a newer build failed. Candidates then come from a commit the upkeep pass has already failed past. Evidence: `internal/testcover/coveragemap.go:58-60` and the `FailedAt` doc at `:105-107`. Fix: add a `map_build_failure` field (or a note) to the answer when `m.Failure != nil`.
- **F2 — NIT, consistency.** `internal/gatecli/redundant.go:89`: the way-forward text for any `redundancyModule` error says to set the dir of tier0 or tier1 with `lym config gate`. That error also covers "the hub declares neither tier0 nor tier1" (`internal/gatecli/redundancytests.go:274`), where setting a dir is the wrong step. Fix: word the way forward as "declare tier0 or tier1 with its dir at the Go module's root".
- **F3 — NIT, design.** `internal/gatecli/redundant.go:94`: `path.Clean` keeps a leading `..`, so `lym gate redundant ../x` reads a `map.json` outside the maps directory. The input is an operator CLI argument, and the read only succeeds if such a file exists. Fix: refuse a `pkg` that is not `filepath.IsLocal` after cleaning, with a way forward naming a module-relative directory.

No wrong result, swallowed error or inconsistent state turned up in the other branches.

**Tool calls (1 of 6)**
1. Read `internal/gatecli/redundancytests.go:245-300` and grepped `redundancyModule(` callers. The manifest lacked the function's error cases and how the existing redundancy step calls it, which F2 and the `.` package case depend on.
