```yaml
lens: code-scope
outcome: reported
```

Lens code-scope: map each diff change to a decision and each decision to its changes.

**Per change → decision**

| Change | Traces to |
|---|---|
| `--map` flag, `mapMode` on `testRequest`, dispatch in `runTest` (`cli.go`, `run.go:43-50`) | D1 |
| Refusals in `flagShapeRefusal` (`run.go:347-360`) and the `cli_test.go` rows | D1 |
| `--map` in `words()` (`detachfront.go:373`) and the `detachfront_test.go` rows | D1 |
| `testmap.go` rewrite: package set, `prepare` go-json and memory refusals, `mapPackages` | D1, D2, D4 |
| Pipelined build and cancel in `close` and `takeBuild` | D2 |
| `testmappackage.go`: coverage binary, per-test runs, reduce, `fillMapTests` | D3 |
| `mapSideBySide`, per-tier batches, `runBatch` alone-rerun, `passed_alone` | D4, D5 |
| `checkClean`, `dirtyOwnedFile`, `dirtyModuleFile`, `endJob`, `write`, `writeFailed` | D6 |
| `mapSlotAsk`, `holdSlot`, no records, exit-code logic in `runTestMap` | D7 |
| Maps directory and `internal/state` writes | D8, unchanged |
| Deleted `testmapaudit.go`, `testmapwait.go`, `testmaporder_test.go`, planning in `testmapplan.go`, `Pool.TryAcquire` | D9, D2, `PATTERN-test-only-code` |
| Deleted `testselect.go`, `declcompare.go`, `upkeep.go`, `FailedAt`, `stampDifferences`, `GoRoot`, `ProfiledFuncs` | D9, `PATTERN-test-only-code` |
| `redundant.go` way-forward text and `redundant_integration_test.go` | D9 |
| README, `docs/overview.md`, the package docs, `template.yaml`, the three PATTERN backgrounds, the `daemonwakeups_test.go` row | D9 and the Constraints. Required by Scope, forced |
| `prebuild.go` header, `gatecli/doc.go` and `testcover/doc.go` rewrites | Forced edits |
| `cli_test.go` `--map` rows | D1 |
| `TestMapSideBySide`, `TestMapSlotAsk` | D4, D7 |
| `blockkey.go` `ProfiledFuncs` removal | Forced by retiring the self-check |

**Per decision → changes**
- D1 to D9: each is built by the changes above; none is left with no change.
- Constraints:
  - `PATTERN-gate-slot` is covered by the third-place bullet.
  - The `PATTERN-daemon-wakeups` line and the test row go together.
  - `PATTERN-test-speed` has its coverage-map section rewritten.
  - The `PATTERN-gate-slot` removal of the testmap holder from the list in `internal/gateslot/doc.go` is covered.

**Findings**

1. **LOW, scope** — `--tags` is refused in map mode, but D1's list omits it.
   - Location: `internal/gatecli/run.go:353`, with a row at `internal/gatecli/cli_test.go` and a line in `doc.go`.
   - Evidence: D1 lists `--tier`, `--upto`, `--gate`, `--guards`, `--stop-on-fail`, `--dir`/`-C` and words after `--`.
   - Note: the plain `--tags` refusal already exists, so the addition only makes the `--map` message name it. It is a small unlicensed extension of the refusal list.
   - Fix: either leave it with the explanation, or drop `--tags` from the map list and rely on the generic `--tags` refusal.

2. **LOW, scope** — `cli.go` help text drops two lines the diff has no decision for.
   - Location: `internal/gatecli/cli.go`, the hunk near line 78.
   - Evidence: removed lines are "Inside a slot already held by an enclosing gate run it runs without acquiring another." and "Each tier prints a summary line to stderr and writes its record and log under the gate directory."
   - Evidence: examples `--gate finalize` and `--upto 2` are also removed.
   - Note: no decision licenses removing these existing help lines, and they remain true for plain calls.
   - Fix: restore the two sentences and the two examples, or record why they went.

I found no other change that traces to nothing. There are no `Pattern-Departure:` trailers in the range.

**Tool calls (2)**
1. `git diff` of the pattern, README, gateslot doc and template, dobbyengine doc and `daemonwakeups_test.go` files. The earlier output was truncated before these.
2. `git diff` of `blockkey.go`, `detachfront_test.go` and `cli_test.go`, plus a search of the commit messages for `Pattern-Departure:` trailers. My context had only the non-test diffs.
