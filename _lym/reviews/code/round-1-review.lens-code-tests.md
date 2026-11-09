```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests lens)**

| Behaviour | Pinning test | Verdict |
|---|---|---|
| Count starts at the first counted landing, so the baseline read counts none | `TestTestMapStep` "every test_map_every landings…" (first step baselines `old`, count is 1 after one landing) | pinned |
| Start after every `test_map_every` landings, with `--max` = `test_map_batch`, count reset, log line | same subtest (starts `[3]`, count 0, exact log line, then a second refresh) | pinned |
| `test_map_every: 0` never refreshes | `TestTestMapStep` "test_map_every 0 starts none" | pinned |
| No start while a refresh is live, a restarted daemon included; count kept; start on the first tick after it ended | `TestTestMapStep` "a live refresh holds the start across a restart…" | pinned |
| A failed start keeps the count, surfaces the wrapped error, and is retried | `TestTestMapStep` "a start that fails keeps the count…" | pinned |
| Nil seams (a beam's dobby) count nothing and start nothing | `TestTestMapStep` "a dobby without the refresh seams…" (asserts count 0 only) | pinned |
| `TestMapRunning` reads only `RoleTestMap` records under `HubOwner` | `TestTestMapRunning`, 4 rows | pinned |
| `endDobbyRoles` ends the dobby, then the refresh for the prime only, and stops on a dobby-end failure | `TestEndDobbyRoles`, 3 rows | pinned |
| Config defaults, `test_map_every` of 0 and a negative value, `test_map_batch` below 1 | `TestLoadConfig`, 3 new rows plus `defaultsWith` | pinned |
| Tick skips `testMapStep` while an anchor-move record stands | none | **gap** |
| `startTestMap`: argv `gate testmap --max N`, `Dir`, `ChildRecord` registration, end on a failed registration | none | **gap** |
| `hubDobbyDeps` fills the two seams for the prime only | none | **gap** |
| `countLandings` adds nothing to the refresh count while a tidy agent is in flight | none | **gap** |
| Error from `TestMapRunning()` or `LoadConfig` returned from `testMapStep` | none | **gap** |
| `endDobbyRoles` surfaces an error from the second (`RoleTestMap`) end | none; `failRole` only covers `RoleDobby` | **gap** |
| `wireCLI` passes `runproc.EndRole` and the real owner to `endDobbyRoles` | none | **gap** |

**Findings**

1. **MEDIUM, design.** `internal/beamcli/dobbydeps.go:565-593`. `startTestMap` has no test. The record's Stop criterion and the spawn rule both rest on it: the `--max` argument, the `RoleTestMap` registration under `HubOwner`, and ending the child when registration fails. `selfexec.ResolveLymBinary` over an injectable lookup lets a test pass a stand-in binary, so this is reachable without launching `go test`. The only evidence is `testmap_test.go`, which fakes `StartTestMap`.

2. **MEDIUM, design.** `internal/dobbyengine/loop.go:644-651`. No test shows that the test-map step is skipped while an anchor move stands. `doc.go` and the `tick` comment both claim it. `loop_move_test.go:31` already sets `MoveRecords` for the tidy step; a refresh-skip assertion belongs beside it.

3. **LOW, design.** `internal/beamcli/dobbydeps.go:217-219`. Nothing pins that only the prime's `hubDobbyDeps` fills `TestMapRunning` and `StartTestMap`, and that a beam's leaves them nil. `dobbydeps_integration_test.go:58-73` already builds both the beam and the prime deps and could assert it.

4. **LOW, design.** `internal/dobbyengine/tidy.go:143-145`. The new count is not pinned to stay still while `s.Tidy != nil`. This is the bound `doc.go` states ("no landing is counted"). The early return above it would hold it, but no new row pins it for the new field.

5. **LOW, design.** `internal/orchcli/beamdown_test.go:25-33`. `TestEndDobbyRoles` has no row where the `RoleTestMap` end fails. The Stop focus item asks that a failure there is surfaced, and the code does return it. A `failRole: RoleTestMap` row with `wantErr` would pin it.

6. **LOW, design.** `internal/dobbyengine/testmap.go:23-35`. The error returns from `LoadConfig` and `TestMapRunning` are untested. Other steps' wrapped errors are covered similarly. A row would be cheap.

I raised no redundancy finding. The new tests are table-driven or subtests and none duplicates an existing test.

**Tool calls**
1. `grep` for `hubDobbyDeps` and `MoveRecords` in `*_test.go` under `beamcli` and `dobbyengine`: to see whether tests already cover the prime-only wiring and the anchor-move skip. My context had neither.
