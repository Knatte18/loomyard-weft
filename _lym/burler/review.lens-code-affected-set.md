```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set, covering what the diff forces outside the lines it touches.

**Affected set per change**

| Change | Set found | Status |
|---|---|---|
| `testmap` verb retired, `--map` added | `README.md`, `docs/overview.md` verb mentions, `gatecli`, `gateslot` and `dobbyengine` docs, `template.yaml`, `redundant` help and refusals, the three PATTERN backgrounds, the `daemonwakeups_test.go` row | covered |
| | log messages in `testmappackage.go` | **not covered** (F2) |
| `testcover` planning deleted | `testcover/doc.go` | covered |
| | `docs/overview.md` tree line for `internal/testcover` | **not covered** (F1) |
| Map run builds no `lym` | `prebuild.go` header, `gatecli/doc.go` map section | covered |
| | `template.yaml` `redundancy_build_timeout_min` comment | **not covered** (F3) |
| Whole rebuild, no kept measured entries | `testmap.go` and `testmapplan.go` headers | covered |
| | `fillMapTests` | **not covered** (F4) |
| `test` help text | the 2000-byte `maxLongBytes` cap in `cmd/lym/clitree_test.go` | within the cap by trimming other help (F5) |
| `testRequest.words`, `mapMode` | detach key and replay; `rungate.go` request, which leaves `mapMode` false | covered |
| Slot ask and multi-slot hold | `PATTERN-gate-slot`, `PATTERN-test-speed`, `gateslot/doc.go` | covered |
| Envelope keys and exit code | the `gatecli` tests and `doc.go`; no other parser found | covered |
| `redundant` way forward | refusal rows, help, comment, `doc.go` | covered; no stencil or contract quotes the text |
| `TryAcquire`, `ProfiledFuncs`, `FailedAt`, `GoRoot`, wait file | docs, tests, wait list | covered |
| Declared PATTERN changes | none declared | n/a |
| `Pattern-Departure` trailers | none in the range | n/a |

**Findings**

**F1. MEDIUM, design**
- Location: `docs/overview.md:407`
- Summary: the `internal/testcover` tree line still says the package "compares a map with a commit to plan which tests a build re-measures or to name the miss that rebuilds the package whole". The diff deletes that selector and upkeep planning (`upkeep.go`, `testselect.go`, `declcompare.go`), so this live doc describes code that no longer exists.
- Evidence: `docs/overview.md:407`, found by search; the deletion in commit `090d0f6d2`.

**F2. LOW, consistency**
- Location: `internal/gatecli/testmappackage.go:562` and `:565`
- Summary: both `logger.Warn` messages still open `gate testmap:`, naming the retired verb in the map run's log, which is its only record. Every refusal of the run was reworded to `gate test --map:`.
- Evidence: `internal/gatecli/testmappackage.go:562`, `:565`.

**F3. LOW, consistency**
- Location: `internal/gateslot/template.yaml:5`
- Summary: the `redundancy_build_timeout_min` comment still bounds each "coverage, mutant and lym build of a coverage map run, …". A map run now builds no `lym` (D3, card 3), and the diff left this line untouched.
- Evidence: `internal/gateslot/template.yaml:5`; `internal/gatecli/doc.go:1101` ("The call builds no `lym`").

**F4. LOW, design**
- Location: `internal/gatecli/testmappackage.go`, `fillMapTests` (comment near line 590)
- Summary: with every job a whole rebuild, the `m` passed in holds only spawn-marked entries, with no `Blocks` or `Funcs`. The kept-entry re-indexing (`reindexed` over `test.Blocks` and `test.Funcs`, the `funcIndex` seeding from kept entries, the `table == nil` fallback to `m.Blocks`) can no longer act on anything. The doc comment still promises it, including that a kept entry's "keys must lie in the fresh tables". This is leftover from the removed incremental planning.
- Evidence: `mapPackage` builds `m := testcover.Map{Commit, Files, OtherFiles}` and appends only spawn entries; `measureTests` calls `fillMapTests(&amp;p.m, …)`; both in `internal/gatecli/testmappackage.go`.

**F5. NIT, scope**
- Location: `internal/gatecli/cli.go`, `testCmd.Long`
- Summary: to add the `--map` paragraph, the diff dropped help facts unrelated to it: what a `--gate` key of scope `all` or `diff` runs, that a call inside a held slot acquires no other, and that each tier writes a summary line, record and log. Card 1 asked only for an added paragraph. The 2000-byte cap is the likely cause.
- Evidence: the `cli.go` diff; `cmd/lym/clitree_test.go:169` (`maxLongBytes = 2000`).

Outside my lens: commits 3 to 5 carry no `Loosened-Check:` trailer although base tests changed, for example `PrebuiltLymIsBuiltOncePerCall` and the `WhileTheLymBuildHangs` row. That is for the tests lens or the handler to judge.

**Tool calls**
1. Repo-wide search for the retired verb, lock and audit wording: the earlier name search was cut off at 80 lines.
2. Search of docs and packages for planning, selector and `lym` build wording: the manifest held only diff hunks, not untouched lines.
3. Search for the removed help sentences and for coverage-map references in contracts and stencils: neither was in context.
4. Search for leftover planning comments, the `overview.md:407` tail and `maxLongBytes`: all untouched lines outside the manifest.
