# Card 4 — End-to-end wedge scenario test

**Create:**
- `internal/websterengine/parentmerge_test.go`

**Uses:**
- `internal/websterengine#RecordBatch`
- `internal/websterengine#BeginBatch`

**Intent:**
Pin the observed wedge (run `batten-hub-ready`, batch 11) end to end, so a regression in any one of begin-batch, record-batch or the head rule shows up as the scenario it breaks.

A new integration-tagged (`//go:build integration`) test file in package `websterengine_test`, reusing the package's existing test fixtures rather than building new ones: `newBeginFixture` for the two-batch plan, scratch repo and geometry, and a `recordFakeEngine` scripted with one fork transcript for the `RecordDeps` built from the same `State`, `Plan`, `Batches` and `Geom`.
The scenario:
1. `BeginBatch(deps, 1)` succeeds and records batch 1's `StartSHA`.
2. The fork's commit lands on the scratch repo and an OK report naming that commit's SHA is written to the reports dir.
3. A parent-side commit off `StartSHA` on a side branch is merged `--no-ff` into the scratch repo's branch.
4. `RecordBatch(recordDeps, 1)` succeeds: batch 1 is terminal, its `CardSHAs` is the fork's commit, and the result carries the moved-HEAD warning naming the merge SHA.
5. `BeginBatch(deps, 2)` succeeds and records batch 2's `StartSHA` as the merge commit's SHA.

The test asserts each of those values exactly.

**Commit:** 4: websterengine: end-to-end test for a parent merge between fork commit and record-batch
