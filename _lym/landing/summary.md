# Record the first recurring test-time and growth audit

Appends a section to `docs/benchmarks/test-suite-timing.md` with the first audit of suite wall time and per-package test counts, measured on `tier0`, `tier1` and `tier2`, each as the median of three warm `-count=1` runs after a discarded cold one.
No code changes and no board task is filed.

## What the section records

- Walls of 18.8 s (`tier0`), 58.8 s (`tier1`) and 50.0 s (`tier2`), with the load and the quiet-machine checks.
- Top-level test counts of 3734, 902 and 69 against the `test-tiers` after-state, tag-only (+905, +223, +4).
- The packages that grew most since the last per-package baseline, and prune-task candidates as lines (`internal/boardengine/boardtest`, `internal/shell`, `internal/battenrecipe`, and a 40 s `tier2` test in `internal/reedengine`).

## Design decisions

- `go run ./cmd/testtiming` is refused to an agent session, so the numbers come from `lym gate test --tier N --all -- -count=1`, the route the refusal names.
  The section says so and lists what that changes: `-p=4` instead of 12-thread parallelism, and a `tier1` wall that includes the build and four vets.
  The older walls are therefore not comparable one to one, and the test counts are the signal.
- The gate's `go test` has no `-count=1`, so a first attempt measured Go test-cache replays.
  The audit keeps the explicit `-count=1` and discards that attempt, which the section records.
- A repeat on an unchanged tree is carried by the gate's record, so one untracked file kept the tree dirty during the measurement and was removed after.
- The decision record's D3 mapped `tier1` and `tier2` to the git-spawning part of the older Tier 2.
  `lym config gate --print` shows `tier1` is the old `integration` tier and `tier2` the old `tmux` tier, and the section follows the config.
- Per-package baseline: `docs/benchmarks/test-redundancy.md`, regenerated 2026-10-27 under older tags, since the timing file holds per-tier totals only; the section marks the comparison indicative.
