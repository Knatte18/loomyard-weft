# gate testmap plans each map at HEAD, keeps what holds, checks the rest

`lym gate testmap` used to rebuild a package's coverage map whole whenever the package's own Go files changed, when the map passed `test_map_max_age_days`, or after a failed build once HEAD moved.
A change to a linked package left the map stale until the age limit caught it.
Now each pass reads HEAD once, asks the coverage selector what changed since each map's commit, and keeps every entry that still holds.

## What changes

- **Plan at HEAD.** With no package named, the verb first plans every package under its build lock, building nothing and holding no gate slot.
  It runs `testcover.Selector` with HEAD as target, the same call the coverage scope makes with the working tree, and hands the selection to the new `testcover.PlanUpkeep`.
  - *Current* (no affected test, no miss): the map moves to HEAD with HEAD's file hashes and every entry.
  - *Affected*: the map moves to HEAD without the affected tests' entries, and those tests are queued.
  - *Miss*: the map stays as it is and the package is queued for a whole rebuild.
    The plan counts misses by reason.
  - Any declared tier0 or tier1 test left with no entry is queued.
    A test the spawn scan marks gets an entry with its verdict instead, and is never run.
  - An outcome entry is dropped and re-queued only when one of the package's own files changed.
  - A package whose build lock another call holds is skipped and counted as queued.
- **Measure the queue.** Jobs run in package path order under `--max`; the rest count as queued.
  A whole rebuild starts from an empty map at HEAD.
  Any other job runs its queued tests into the map the plan wrote, replacing their entries.
  Every run now enters the map.
  A failed, timed-out, skipped or empty run records its `TestOutcome`, and each entry carries the `Commit` and `MeasuredAt` it was measured at.
- **Self-check.** A job that keeps entries checks them against the fresh build.
  An entry whose block keys are missing from the fresh profile, or whose linked functions are missing from the linked packages' functions (`FuncIndex.ProfiledFuncs`), is re-run in the same job and counted.
- **Failed builds keep the map.** A failed build keeps a map that has test entries, with its commit, entries and marker.
  The failure and HEAD go into the new `Map.Failure`.
  A package with no such map still gets the failure-only map with `BuildFailure`.
  `Map.FailedAt` reads either form, and no pass at that HEAD queues the package again.
- **Audit.** Once nothing is left queued, the pass re-measures the `testMapAuditTests` (3) oldest-measured passing entries of the maps that stand at HEAD.
  It runs only if the pass already holds a slot or the new `gateslot.Pool.TryAcquire` gets one at once.
  A fresh result that differs from the kept one counts as an audit mismatch.
  Every re-measured entry is replaced, so the next round reaches other entries.
- **`test_map_max_age_days` is retired.** It is gone from the template and `Config`, and is listed in `RetiredKeys`, so a hub file that still sets it loads and reconcile prunes it.
- **Exit record and report.** `testcover.ExitRecord` replaces `Unbuilt` with:
  - `PlannedHead` and `Queued`;
  - `Plan`: current, affected, and misses by reason;
  - `SelfCheckMismatches` and `AuditMismatches`;
  - `Packages`: build seconds, run seconds and test count per package.

  The verb's envelope prints the same fields.
- **Redundancy step.** Only passing entries count as covers; an outcome entry or a spawn-marked entry covers nothing.
  A map that records a later failure beside its entries is judged by those entries.

## Design decisions

- `PlanUpkeep` lives in `internal/testcover` as a pure function, so the plan's rules are pinned by a tier0 table test (`TestPlanUpkeep`).
  `internal/gatecli` reads the trees, writes the maps and runs the jobs.
- `MapFormatVersion` stays at 2.
  The new fields are additive and existing maps still read.
  An older map's timed-out or failed tests have no entries, so the first pass queues them once and records their outcome.
- A test the runner could not start is recorded as failed, so it is not re-queued on every pass.
- The audit takes only maps that stand at HEAD and record no failure there: its build is of HEAD, so re-measuring an entry of an older map would mix commits.
- The audit size is a named constant (`testMapAuditTests`).
  `testMapRequest.auditTests` carries it so the integration tests can turn the round off or size it.
- Named packages are still rebuilt whole, unplanned.

## Not in this change

- D12's acceptance line says the dobby reads the exit record.
  Nothing in the dobby reads it yet.
  The dobby's decision to start a pass on queued work is `testmap-upkeep-dobby`'s (D8), so that read is left to it.
- The slot-wait rule for planning a locked map again (D9) belongs to `testmap-upkeep-runs`.
  Here a locked map is skipped and counted as queued, so the next pass plans it again.

## To check by hand

- The `lym gate testmap` integration tests in `internal/gatecli/testmap_integration_test.go` and the redundancy integration test are tier1 and were not run.
  Their expectations were reworked for the new verdicts: outcome entries, spawn entries without coverage, `Queued`, kept maps on a failed build, the re-stamp and affected cases, the self-check and the audit.
  Only `go vet` was run over the touched packages.
- `plan` counts on the integration subtests assume the fixture module's package set (five packages in the linked fixture).
