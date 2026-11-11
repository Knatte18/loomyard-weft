```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the testmap upkeep diff. Each function, the case I ran, and whether it was correct:

- `PlanUpkeep`, Current at an older `Commit`: the map re-stamps with every entry and `Changed` is true. Correct.
- `PlanUpkeep`, Miss: the map is left alone and nothing is queued. Correct.
- `PlanUpkeep`, Affected: `TestHit` is dropped and queued, while `TestFailed` is kept on a linked-only change. Correct.
- `PlanUpkeep`, own file changed: outcome entries are dropped and queued. Correct.
- `PlanUpkeep`, spawn-marked test with no entry: it gets an entry with its verdict and is never queued. Correct.
- `PlanUpkeep`, entry whose test is no longer declared: the entry is kept forever. **Finding F1.**
- `PlanUpkeep`, Current re-stamp of a map whose `Failure` names an older commit: the stale `Failure` is carried along. **Finding F2.**
- `planPackage`, locked package: it is skipped and counted as queued. Correct.
- `planPackage`, `FailedAt(head)`: nothing is queued and no verdict is counted. Correct.
- `planPackage`, `ReadMap` error: the zero map makes the selector return a miss, so a whole rebuild is queued. Correct.
- `planPackage`, `mappedTests` or spawn-scan error: it queues a whole rebuild, whose own scan then records the failure. Correct.
- `mapPackages`, cap reached: the remaining jobs count as queued. Correct. Audit runs only when `queued` is 0. Correct.
- `mapPackages`, a job skipped before its build: `measured--`. Correct.
- `mapPackage`, whole job with no mapped test: it writes an empty map at HEAD. Correct.
- `mapPackage`, non-whole job whose map is not at HEAD: it skips and counts the package as queued. Correct.
- `mapPackage`, self-check when no run wrote a profile: it is skipped. **Finding F3.**
- `mapPackage`, a skip after the tests ran: it appends to `r.measures` before the dirty and changed-under-tests checks, so a skipped package still shows build and run seconds. It also leaves the plan-written map with its gaps. The gaps are requeued next pass, so only the report is off. Included in F3 below.
- `selfCheck` and `fillMapTests`, kept entries re-indexed by block key and linked function: correct as long as the self-check ran, since a missing key would silently map to index 0.
- `runTests` and `runTest`: a cancelled context returns an error and records nothing, and a runner error is recorded as failed. Correct.
- `writeFailed`, map with entries: it keeps the map and marker and sets `Failure` at HEAD, and it writes with `WriteMap` so the marker is unchanged. Correct.
- `writeFailed`, no entries: it writes the failure-only map. Correct.
- `writeFailed`, called for a self-check or key error that is not a build failure: it records that error as a failed build at HEAD, which blocks requeue at this HEAD. Correct per D5, but it is the same effect as F3's skipped check, so it is not a separate finding.
- `audit`, candidate order: oldest `MeasuredAt` first, falling back to `m.BuiltAt` for legacy entries. **Finding F4.**
- `audit`, slot: `TryAcquire` returns false when none is free, with no wait. Correct.
- `audit`, mismatch count: `entryCoverage` compares outcome, block keys and linked functions between `before` and the new map. Correct.
- `TryAcquire`, drain lock held: it returns a nil lease and no error. Correct.
- Selector at HEAD: `headSelector` stamps the environment and the selector adds go.mod and go.sum through `StampModuleFiles`, matching the stamp `writeMap` writes. Correct.

## Findings

### F1 — MEDIUM, design
Location: `internal/testcover/upkeep.go` (the loop over `in.Map.Tests`; `in.Declared` is used only for the queue).
Summary: `PlanUpkeep` keeps the entry of a test no longer declared in a mapped tier at `Commit`, because it drops entries only for affected tests and outcome entries.
Evidence: `testselect.go:205` and `testselect.go:310` show the selector reports only changed or new test declarations (`own.tests`), so a deleted or re-tiered test is never dropped by a Current or Affected plan. Before this diff, the whole rebuild that every own-file change triggered dropped such entries.
Effect: the stale entry stays through every Current verdict. The redundancy step reads it, and the selector can select a test that no longer exists.
Fix: in `PlanUpkeep`, also drop an entry whose name is not in `in.Declared` and set `Changed`. Add a `TestPlanUpkeep` row.

### F2 — LOW, design
Location: `internal/testcover/upkeep.go` (`m := in.Map` carries `Failure`).
Summary: a Current or Affected plan that moves a map to `H` keeps a `Failure` naming an older commit.
Effect: the stale record stays until a measuring job clears it (`m.Failure = nil` in `mapPackage`). A Current map with no queue keeps it forever, and the exit record and docs describe it as a failure "since".
Fix: when `in.Map.Failure` is set and its `Commit` is not `in.Commit`, set `m.Failure = nil` and `plan.Changed = true`. Alternatively state in `Map.Failure`'s comment that a stale one is tolerated.

### F3 — LOW, design
Location: `internal/gatecli/testmappackage.go`, `if profile := firstProfile(runs); !job.whole && profile != nil`.
Summary: D7's self-check runs only when some run wrote a profile.
Effect: in a job where every queued test fails, times out or is skipped, no profile exists, so the kept entries are never checked against the fresh build. The table is also not replaced, so the map stays internally consistent but unverified. The `r.measures` append ahead of the dirty and changed-under-tests skips is the same kind of report error.
Fix: for the self-check, obtain the fresh block table without a test profile, or record that the check is skipped for that job and count it. Move the `r.measures` append after the skip checks.

### F4 — LOW, design
Location: `internal/gatecli/testmapaudit.go:42-45`.
Summary: a legacy entry with no `MeasuredAt` takes `m.BuiltAt`, which every re-stamp resets through `writeMap`.
Effect: legacy entries look younger after each re-stamp, so the audit's "oldest measurement first" order is not what D7 asks for.
Fix: treat a zero `MeasuredAt` as the oldest, `time.Time{}`, instead of `m.BuiltAt`.

Tool calls:
1. grep in `internal/testcover/testselect.go` for the selector's Select, `changedTests` and stamp handling, because the manifest lacked the selector's code.
2. Read `testselect.go` lines 168-247 for `Select`, `mapMiss` and `environmentMiss`, which I needed to check the environment stamp and the Format marker check.
3. grep `testselect.go` for how `own.tests` is built, to check whether deleted tests are reported; it was not conclusive, so F1 rests on the visible `PlanUpkeep` loop.
