```yaml
lens: code-affected-set
outcome: reported
```

The affected set of the diff's changes is complete, and I raise no findings.

**Per change, the set found:**

- **`Store.UpsertTask` clones `fields` before the checks (store.go:671-675).**
  - Board wrappers that call it: `Board.UpsertTask` (board.go:225), `UpsertTaskReporting` (siblings.go:153), `UpsertTaskWithRecord` (recordupsert.go:59), `FileFrontTask` (front.go:63), `fileRaddleEntry` (raddlefile.go:187).
  - Store callers: intake import and fold (intake.go:261, 301), and `MergeTasks`' final call (store.go:1350).
  - External callers: boardcli upsert (cli.go:635/637), beamcli FileTask (deps.go:277), retrocli (cli.go:163), boardserve applyWrite (write.go:134), fronttask via `Create` (fronttask.go:202).
  - Covered: none of these reads the map after the call, expecting a key dropped or added.
  - `UpsertTaskReporting` reads `fields["slug"]` before the call, and `FileFrontTask` reads the slug before its own front and on_land refusal. Neither depends on the store mutating the map.
- **`Store.UpsertTasksBatch` clones each element (store.go:1072-1077).**
  - `Board.UpsertTasksBatch` (board.go:343) and boardcli upsert-batch (cli.go:717), which uses only `len(tasks)` afterwards. Covered.
- **`Store.MergeTasks` moves its clone to the top (store.go:1259).**
  - `Board.MergeTasks` (board.go:322) and boardcli merge (cli.go:1079), which never reads `upsertFields` afterwards. Covered.
  - The `issues`, `created` and `project` keys were already written to a clone before this diff, so no caller saw them.
- **Doc comments.**
  - The check helpers' "dropped from fields" wording (decision.go:173, front.go:21, owners.go:117, project.go:49) still holds: each drops from the map it is handed, which is now the clone.
  - doc.go:32 and doc.go:88 say "dropped as a no-op", which still holds.
  - The Board wrappers' docs make no claim that the store mutates the map. Covered.
- **Tests.** No test asserts that a key was deleted from, or added to, a caller's map after these calls.
  - The grep hits at mergeparent_test.go:139, fronttask_test.go:61 and raddlefile_test.go:50 read maps the test's own fakes or callers built. They do not read the store's effect.
- **Repository rules.**
  - No stencil or spec is touched. No PATTERN changes are declared, and none land.
  - `PATTERN-tolerant-records` and `PATTERN-test-speed` are not breached: the test keeps `t.Parallel`.
- **Commit message.**
  - It carries a `Root-Cause:` trailer naming the in-place deletion. There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.

**Findings:** none.

**Tool calls:**
1. A grep of the boardengine docs for "dropped from" and "payload map" wording, and a list of the test files calling the three methods. My context lacked the doc.go wording and the test call sites.
2. A grep of the tests for reads of dropped or merge-added keys from caller maps, and of PATTERN-tolerant-records for clone or payload rules. My context lacked whether any test relied on the old mutation.
