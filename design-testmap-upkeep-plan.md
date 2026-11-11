# lym gate testmap plans each map at the prime's HEAD, keeps what still holds, and checks what it keeps

`testmap-upkeep-plan` · Task · feature, testkit, gate, dobby · running · Seed-Child

Decision record: [Sketch](decision-testmap-upkeep-plan.md)

Depends on: [`testmap-selector-upkeep-rules`](design-testmap-selector-upkeep-rules.md)

Filed from the Sketch `testmap-upkeep-design`.

Takes: testmap-selector-upkeep-rules' selector with a commit target and its environment stamp; today's lym gate testmap (runTestMap, mapPackages, stale, mapPackage, writeMap) in internal/gatecli, with its blocking slot take and one process per test
Delivers: the plan at H with the Current, Affected and Miss verdicts, re-stamps, marker updates and queued gaps, under the build lock with held locks skipped (D1); outcome entries, spawn verdicts and nothing recorded for an ended run (D4); the kept map with a failure field, and no requeue at the failed HEAD (D5); test_map_max_age_days retired from gateslot, the self-check on every build and one audit round per pass (D7); testcover.ExitRecord's planned HEAD, queued count, verdict counts with miss reasons, mismatch counts and per-package build and run seconds, and the verb's report of them (D12); the redundancy step counting only passing entries as covers and judging a map with a failure field by its entries
Exports: the plan and queue a pass works, the job that measures a queued test list, testcover.ExitRecord's new fields

The part changes these packages:

- internal/gatecli (about 850 changed lines)
- internal/testcover (about 350 changed lines)
- internal/gateslot (about 30 changed lines)
