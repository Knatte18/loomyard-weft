```yaml
lens: code-correctness
outcome: reported
```

Task restated: review the code-correctness lens of a docs-only diff.

**List (each function or branch, case run, correct or not)**

The diff changes no function, branch or error path. It adds one README index line and four markdown files under `docs/research/speed/testmap-upkeep-2026-11-11/`. `git diff --stat` lists nothing outside `docs/`. No code case can be run, so the list is empty.

I did check the report's code-facing claims, since they are the only checkable correctness content in the diff:

| Claim in the diff | Case run | Result |
|---|---|---|
| `ExitRecord.Unbuilt` is written and never read by production code | grep for `Unbuilt` in `internal` and `cmd` | Correct. The only reference outside the field's definition is `internal/gatecli/testmap.go:482`, a write. The other hits are the unrelated raddleparser `CheckDecisionUnbuilt` names. |
| `testMapStep` starts a refresh after `test_map_every` landings, with `--max` set to `test_map_batch` | read `internal/dobbyengine/testmap.go` | Correct. `landings < cfg.TestMapEvery` returns early, and the call is `StartTestMap(cfg.TestMapBatch)`. |
| Defaults `test_map_every: 5`, `test_map_batch: 5`, `test_map_max_age_days: 7` | grep the two template files | Correct. `internal/dobbyengine/template.yaml:4-5` and `internal/gateslot/template.yaml:4`. |
| `slots: 2` in the template, 3 in this hub's `gate.yaml` | grep | Correct. Template line 1 holds 2, and `../_bolt/_lym/config/gate.yaml:1` holds 3. |
| `Pool.tryAcquireAny` is unexported, while `Holders` and `Draining` are exported | grep `internal/gateslot/pool.go` | Correct (lines 301, 338, 374). |
| `writeFailed` writes a map with no tests | grep `internal/gatecli/testmappackage.go:274` | Consistent with the claim. I did not read the body to confirm it overwrites a previously usable map. |
| testmap never calls `ScanSpawns`, only the redundancy step does | grep | Correct. The only production call is `internal/gatecli/redundancy.go:259`. |
| `tiergate.Require` call sites: 4424 for tier 0, 903 for tier 1 | grep count | Correct. Matches exactly. |
| 172 first-parent commits on main from 2026-11-04 to 2026-11-11 | `git log --first-parent --since --until` | I counted 171. The report's own figure differs from O's 173, and the gap to mine follows the date boundary, so I raise no finding. |

**Findings:** none. Severity none, no class.

**Tool calls:** none. I read nothing beyond the parent context, which already held every file I cited.
