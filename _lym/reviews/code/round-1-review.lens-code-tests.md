```yaml
lens: code-tests
outcome: reported
```

**Task:** lens code-tests. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test |
|---|---|
| `drift.yaml` loads `scope: full` | `TestLoadConfig` "scope full loads" |
| `drift.yaml` loads `scope: partial` with its `skip` list | `TestLoadConfig` "scope partial loads with its skipped tests" |
| `scope: off` loads | None. No row sets `scope: off` in `LoadConfig`. It goes through the same arm as full, and the baseline test covers `off` downstream, so this is a gap in the config table only. |
| Unknown scope is refused, naming `"scope"`, `run_settings` and `lym config drift` | `TestLoadConfig` "an unknown scope" |
| Partial with no skipped test is refused | `TestLoadConfig` "scope partial with no skipped test" |
| Skip under another scope is refused | `TestLoadConfig` "a skipped test under scope full". The empty-scope arm and the `off` arm share its `case`, and no row sets `skip` with an empty scope. |
| `Config.Skip` is set only under partial | Pinned by the partial row's `want`. The other direction is unreachable, because `checkScope` refuses skip elsewhere. |
| `scope` and `skip` are accepted in the local layer, which Seed-Child writes from `run_settings` | `TestLoadConfig` "a local scope with its skipped tests loads over the board's" (`config_test.go`). `TestModules_EveryTemplateKeyClassified` (`configreg_test.go:80`) pins the `CategoryNone` registration. |
| A configured `full` is recorded, `baseline.md` is written, no agent runs and the capture starts | `TestDriftBaseline` "a configured scope full…". It asserts `wantAgents` (default 0), `wantStarts` 1 and the commit set through the table loop at `baseline_test.go:497`. |
| A configured `off` is recorded, no agent and no capture | `TestDriftBaseline` "a configured scope off…" (`wantStarts` 0) |
| A configured `partial` records its skip list and `baseline.md` names the skipped tests | `TestDriftBaseline` "a configured scope partial…" |
| A scope already recorded wins over the configured one, and the missing `baseline.md` is written for it | `TestDriftBaseline` "a recorded scope wins…". `seedScopeRecord(..., false)` writes no report, so this row also pins the missing-report write on the kept-scope path. |
| An existing `baseline.md` is not overwritten when a scope is configured | **None.** The row at `baseline_test.go:286` ("a scope record with baseline.md spawns no agent") sets no `configScope`. The new rows never seed a report. Without the `os.Stat` early return in `settleConfiguredScope` (`baseline.go:150`), the agent's or a parent's `baseline.md` would be overwritten on every resume and no test would fail. |
| `drive`'s new `d.Config()` error gives a Stuck naming `lym loom resume` | **None** for the `drive` call itself. The existing `configErr` row fails earlier, at `passWhenSwitchedOff` in `Call` (`baseline.go:62`). The `drive` branch is not reachable from the fixture, because it returns the same config twice. |
| Error paths of `settleConfiguredScope`: the `Stat` error other than not-exist, `MkdirAll` and `WriteFile` | None. These are filesystem failures on a path the same function just created, so I count them as low value. |
| With drift off and a configured scope, nothing is recorded or spawned | No row sets `configScope` together with `disabled`. `Call` runs `passWhenSwitchedOff` before `drive`, and the existing switched-off row asserts no commit and no spawn, so a reordering would still be caught only if the config were set. |
| `configuredBaselineReport` text and the omitted skip line when there is no skip | The text is pinned by `Contains` on "drift.scope" and "Scope: …". The absence of "Skipped tests:" under `full` and `off` is not asserted. |
| `orch-role.md`, `docs/overview.md`, the CLI help, `doc.go` and the template comments | Static prose, exempt. |

**Findings**

1. **LOW, `design`.** `internal/driftengine/baseline.go:150` has no test for the existing-`baseline.md` guard in `settleConfiguredScope`.
   - A configured scope with a scope record and a report already on disk should leave the report untouched.
   - Neither the configured-scope rows nor the recorded-scope row seeds a report, and the "has baseline.md" row (`baseline_test.go:286`) sets no `configScope`.
   - A fix is one row: seed a record with `report=true`, set `configScope`, and assert `baseline.md` is still "expected drift" with no commit side effect.

2. **NIT, `consistency`.** `internal/driftengine/baseline.go:84` adds a second `d.Config()` read in `drive`, and its Stuck branch is untested and unreachable.
   - `Call` has already run the same load through `passWhenSwitchedOff`, which gives the same Stuck.
   - If the branch stays, it needs a row. Since the fixture's config cannot differ between the two calls, the cleaner fix is for `drive` to take the config `Call` already loaded.

3. **NIT, `consistency`.** `internal/driftengine/config_test.go` has no `LoadConfig` row for `scope: off`. The `checkScope` arm that accepts it, and the `off` skip-refusal, are exercised only through the full and partial rows.

I found no duplicate tests among the new rows; each covers a distinct branch.

**Tool calls (2 of 6)**
1. `grep` of `seedScopeRecord` and the spawn-count assertions in `baseline_test.go`. My context lacked the second argument's meaning and whether "no agent" is asserted. It is: `f.seats.Calls` against `wantAgents`, line 497.
2. `Read` of `baseline_test.go:96-128`. My context lacked the fixture wiring of `Seats`, `BaselineTable` and `StartCapture`.
3. `grep` of `f.seats|wantSpawn` in `baseline_test.go`, to confirm line 497 is the only agent-count check.
