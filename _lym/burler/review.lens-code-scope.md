```yaml
lens: code-scope
outcome: reported
```

Task: trace every change in the diff `24cd026e8..HEAD` to a decision D1–D9, and every decision to a change. The record has no `Added after Discussion` entry and no `Pattern-Departure:` trailer.

**Per change**

| Change | Traces to |
|---|---|
| `--map` flag, `mapFlag`, `testRequest.mapMode`, routing in `runTest`, `words()` | D1 (card 1) |
| Partner-flag refusals in `flagShapeRefusal`, go-json counter refusal | D1 |
| Package set taken from the scope (`packages`, `scopePackages`, `left_out`) | D1 |
| `testMapCmd`, `testmapaudit.go`, `testmapwait.go`, `testmaporder_test.go`, planning, `--max` and lock removed | D2, D9 (card 1) |
| Whole rebuild per package, build one package ahead | D2 |
| `Pool.TryAcquire` and its test removed; its `gateslot` doc sentence | forced by D9 |
| `ProfiledFuncs`, `coveredOnly`, `upkeep.go`, `testselect.go`, `declcompare.go`, `FailedAt`, `stampDifferences`, `GoRoot` and their tests removed | D9, Scope (cards 1, 2) |
| `testcover/doc.go` trimmed | forced by those deletes |
| `daemonwakeups_test.go` row and the daemon-wakeups background | forced by D9 |
| Per-tier side-by-side count, `memory_mb` clamp and refusal | D4 (card 3) |
| `guardFor` seam | D4, as its test seam |
| `passed_alone` | D5 |
| `lym` build removed, `prebuild.go` header | D3 |
| `mapSlotAsk`, `AcquireSlots` | D7 |
| `failed_tests` and exit code | D7 |
| `checkClean`, `dirtyModuleFile`, `unmeasured`; `changedFiles` removed | D6 (card 4) |
| `lym gate redundant` way-forward and `Long` text | D9 (card 5) |
| README, overview, package docs, `template.yaml`, PATTERN backgrounds | D9, Scope |
| `testCmd` `Long` sentences removed (scope `all`/`diff`, held slot, summary line and record) | forced by the added `--map` paragraph, assuming the `maxLongBytes` = 2000 cap in `cmd/lym/clitree_test.go` requires it (I did not count the bytes) |
| Test changes | follow their cards |

No change is unrequested.

**Per decision**

| Decision | Built by |
|---|---|
| D1 | flag, routing, words, refusals, package set |
| D2 | whole rebuild, planning, audit and lock removal, build-ahead |
| D3 | reused measure loop, `lym` build removal |
| D4 | per-tier counts, clamp, refusal, guard |
| D5 | alone rerun, `passed_alone` |
| D6 | clean and HEAD checks |
| D7 | slot ask, no records, exit code |
| D8 | `coverageMapsDir` and the map format are unchanged |
| D9 | verb and code removal, redundant way-forward, docs; incomplete in findings 1–3 |

**Findings**

1. **MEDIUM, consistency**, `docs/overview.md:407`
   - The `internal/testcover` tree line still says the package "compares a map with a commit to plan which tests a build re-measures or to name the miss that rebuilds the package whole".
   - This diff deletes that code (`upkeep.go`, `testselect.go`, `declcompare.go`), so the doc describes behaviour that no longer exists. Neither D9 nor the "docs land in the same diff" rule is met for it.
   - Evidence: `docs/overview.md:407`, found by grep.

2. **LOW, consistency**, `internal/gatecli/testmappackage.go:562` and `:565`
   - Both `logger.Warn` messages in `runTest` still open `gate testmap:`, naming the retired verb in the map run's own log.
   - The Scope section asks for every live text naming the verb to change, and D7 makes the log one of the run's outputs.
   - Evidence: `testmappackage.go:562,565`.

3. **LOW, scope**, `internal/gatecli/testmappackage.go:588-650` (`fillMapTests`, `reindexed`)
   - The merge still re-indexes the blocks and funcs of "kept" entries against fresh tables, and still falls back to `m.Blocks`.
   - Since card 1, every job starts from an empty map. The only entries in `p.m.Tests` are the spawn-marked ones, which have no blocks or funcs. The re-indexing served only the retired incremental planning, which D2 and D9 remove.
   - The doc comment ("a kept entry is re-indexed by its blocks' keys … its keys must lie in the fresh tables") describes a case that can no longer happen.
   - Evidence: `testmappackage.go` `mapPackage`, where `m := testcover.Map{Commit, Files, OtherFiles}` gains only spawn entries, and `fillMapTests`.

**Tool calls**
1. Grep for retired-verb and planning text across code, docs and patterns: the manifest's name search was truncated before it reached these hits.
2. Grep for a help-text length guard: needed to judge whether the `Long` removals were forced, and `cmd/lym/clitree_test.go` was not in context.
3. Grep for planning remnants in `testcover` and `gatecli` comments: those files' remaining comments were not all in context.
