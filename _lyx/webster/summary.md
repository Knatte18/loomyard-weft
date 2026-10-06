# test-suite-prune-core: six test suites pruned and measured

All 17 batches are `done` per `lyx webster status`; batch 10 was a recovery batch.
This session found nothing left to run and forked nothing.
The narrative below comes from the digests recorded by earlier sessions.

## Tooling and baseline (batches 1 to 6)

The redundancy tool gained its single-test flag form, a narrower out-of-process exclusion, the `//testtiming:keep` directive and the per-package report splice.
The before measurement was recorded and the six package sections were rerun.

## Per-package work (batches 7 to 16)

- **shedadapters, shuttleengine, websterengine (7 to 10):** pruned, with websterengine's tests moved off real git first.
- **reedengine (11, 12):** one test moved to the untagged tier, scenario merges removed 3 fixture builds, and 327 tests became 212.
- **loomcli (13, 14):** tests moved off real git and tmux, scenario merges removed 27 fixture builds, and 253 tests became 167.
- **fabricengine (15, 16):** four tests moved to the untagged tier, scenario merges removed 8 and then 26 hub builds, and 688 tests became 610.

## Measurement (batch 17)

The six sections were rerun and all read "No candidates", with no failed test.
`docs/benchmarks/test-suite-prune-core.md` records the after measurement: every package has fewer tests, none got slower, and coverage is equal or up 0.1 points.

## Deviations and caveats

- Batch 16 changed one file outside its declared file-ops: `internal/fabricengine/mergeerrors.go`, a comment now naming the surviving test.
- The batch 12 and 14 keep reasons end in a generic clause that was not checked against every covering test.
- The batch 14 and 16 implementers did not check every deletion or hook-test keep against D4's hook-or-alias rule.
- Batch 17's first timing run showed `reedengine` as FAIL with the test name not captured; it passed in later runs and the report records it undiagnosed.
- `gofmt -l` flags `internal/fabricengine/pull.go`, which no batch touched.
- Batches 15 to 17 ran their own card gates only; the plan-level verify is left to the gate.
