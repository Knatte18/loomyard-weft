# Card 16 — batten-records-integration

**Create:**
- `internal/battencli/records_integration_test.go`

**Uses:**
- `internal/battencli#wireForHub`
- `internal/battencli#battenCLI.wire`
- `plan:internal/battenshed#ChildApproval`
- `plan:internal/shedrun#DriveReportsDir`
- `plan:internal/loomengine#LoomDurableDir`

**Intent:**
One end-to-end check that the whole hand-off keeps the run's records: a batten run whose child goes `awaiting`, gets an approval record, is resumed by batten, reaches `done`, and is torn down ends with the friction notes and the drive report reachable from the archive tag on the weft origin.

`records_integration_test.go` (build tag `integration`, white-box `package battencli` like `lifecycle_integration_test.go`) builds a hubforge hub with a weft origin, wires batten through `wireForHub` over the short-poll recipe, and stubs at the field level after `wire`:
- `InnerRun.ReadStatus` answers `awaiting` at Publish until the approval-driven spawn has happened, then `done`;
- `InnerRun.Spawn` on its first call writes a friction note under the task worktree's `loomengine.LoomFrictionDir` and a drive report under `shedrun.DriveReportsDir`, and commits both with `fabricengine.CommitAnchoredPaths` onto the task pair, standing in for the child's transition commit and `lyx loom commit-records`;
- `InnerRun.DriverAlive` reports not live.
The test writes the approval record through `landingshed.WriteApproval` at the task worktree's `loomengine.LoomApprovalPath` between steps, drives the run to done, and asserts: Spawn ran once for the approval and never again for the same record; the run ends `done` past `Worktree-Teardown`; the task pair is gone; and `git show archive/<slug>/<tip>:<path>` in the weft origin returns both files.

No real provider or driver is spawned, per batten's cost declaration.
The `Spawn` stub's own commit is the one link this test does not exercise: `battencli` cannot reach loomcli's unexported commit seam, so the proof that the real records pathspec commits friction notes, their archive rename and the drive report onto the weft tip lives in card 5's and card 7's integration tests, and this test proves the rest of the chain, from a committed weft tip to an archive tag on the weft origin through batten's own teardown.

This card only adds a test: the behavior it exercises lands in the earlier cards.
