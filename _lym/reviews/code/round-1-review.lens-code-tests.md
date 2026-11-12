```yaml
lens: code-tests
outcome: reported
```

Task: list, per behaviour the diff adds, the test that pins it (or none), for the code-tests lens.

**Per-behaviour list**

| Behaviour | Pinning test |
|---|---|
| `JudgeMap`: keep-marked entry gets `ReasonKept` and is not a cover | `TestJudgeMap_KeptAndUnmeasuredEntriesNeitherJudgedNorCovering` (row `Keep` has block 1, which only `Super` covers; if it counted as a cover, `Super` would be contained) |
| `JudgeMap`: outcome entry gets its outcome as reason | the same test (row `Timed`) |
| `JudgeMap`: spawn-marked entry gets `ReasonOutOfProcess` | the same test (row `Spawned`) |
| `JudgeMap`: containment, removable flag and covering list over map block indexes | the same test (`Sub` and `Super`) |
| `JudgeMap`: verdicts come back in the map's order | the same test (the `want` order interleaves judged and unjudged rows) |
| `JudgeMap`: `Unresolved` spawn marker, no-blocks entry | none, but both are the existing `Judge`'s behaviour (`notJudgedReason`), which `TestJudge` covers. No finding. |
| Verb: candidate list with tier, seconds, removable, covering | `TestGateRedundant/ListsTheContainedTestsAndSetsApartTheKeptAndUnjudgedOnes`. Its candidate is `TestCovSub` at tier 0, so a wrong `Tier` mapping passes because 0 is the zero value. |
| Verb: kept list with the keep reason from `ScanKeeps` | the same subtest (`TestCovKept`) |
| Verb: not-judged list with reason | the same subtest (`TestCovFails`) |
| Verb: JSON envelope fields `package`, `map_commit`, `map_built_at`, `tests` | the same subtest |
| Verb: `./cov` is cleaned to `cov` | the same subtest |
| Verb refuses outside a hub, with a way forward | `RefusesOutsideEveryHub` |
| Verb refuses for a package with no map | `RefusesAPackageWithNoMap` |
| Verb refuses for a map holding only a failed build | `RefusesAMapHoldingOnlyAFailedBuild` |
| Verb refuses for an unreadable map, including `checkMapBlocks` rejecting an out-of-range block index (`redundant.go` switch case `err != nil`) | none |
| Verb refuses for a failed keep scan (`redundant.go`, `ScanKeeps` error branch) | none |
| Verb refuses when the hub's gate config can't be read, or `redundancyModule` fails (no module dir) | none |
| Verb refuses for a cwd-read or hub-resolve error | none |
| Registration of `redundantCmd` in `cli.go` | implied by the subtests, which run `redundant` through `RunCLIIn` |
| Docs (`doc.go`, `overview.md`, `PATTERN-test-speed.md`) | static prose, exempt |

**Findings**

1. **LOW, design, `internal/gatecli/redundant.go:96`, `redundant.go:105-110`.** Two refusals the summary and doc.go promise have no test: an unreadable or out-of-range map, and a failed keep scan.
   - Evidence: `redundant_integration_test.go` has only three refusal rows (no hub, no map, build failure). The commit message and `doc.go` list "unreadable" and "failed keep scan" refusals.
   - Fix: add two rows to the `refusals` table. One writes a map whose test names a block index outside its table. The other writes a `*_test.go` file with a `//testtiming:keep` line that has no reason. Each asserts the error text and "way forward".
   - Other refusals in the table are not covered either: the gate-config and module errors. The `redundancyModule` error is plausibly reachable with a hub whose tier dirs have no `go.mod`.

2. **NIT, design, `internal/gatecli/redundant_integration_test.go:78`.** The candidate row has `Tier` 0, the zero value, so `Tier: test.Tier` is never distinguished from an unset field.
   - Fix: give `TestCovSub` a `Tier: 1` in the written map and in `want`.

No redundancy finding. `TestGateRedundant` is `not judged, out of process` in the report, and `TestJudgeMap_…` is `unique`. The tier1 test's candidate case repeats the `JudgeMap` containment logic, but it pins the verb's mapping to the output shape, the keep reasons and the refusals.

**Tool calls:** none. The inherited context already held every file this lens needed.
