# Round 1 fixer report — plan

## Changes

- **F1 (BLOCKING), fixed as proposed**, in `_lyx/plan/06-loom-hard-cap-config.md`.
  The premise holds: `internal/loomengine/config.go:903` and `:918` carry static "in loom.yaml" texts without `lyx loom resume`, and `ConfigSpec` leaves the new keys settable through `run_settings`.
  - `LoadConfig` bullets: `review_hard_max_bounces` below 1 refuses with `CapRoute(dirs, "review_hard_max_bounces")` and each negative per-perch hard key with `CapRoute(dirs, <that key>)`, both ending "then run \"lyx loom resume\"", and neither reuses the existing loops' static texts.
  - `refusal-spec.md` bullet: the "config: review key below 1" row stays unchanged.
    The new row is now "config: hard cap out of range", covering hard below 1, per-perch hard negative and hard below soft, all with the per-layer way forward and `lyx loom resume`.
  - `TestLoadConfig_Refuses`: the `review_hard_max_bounces` 0 row and the per-perch -1 rows run in a worktree-only config and want `lyx config loom --set <key>=<n>` and `lyx loom resume`.
    The hub-stack subtest adds a local `review_hard_max_bounces: 0` row that wants `--layer local`, `lyx board upsert` and `lyx loom resume`.
- **F2 (LOW), fixed as proposed**, in `_lyx/plan/08-circling-continue-hard-cap.md`.
  The premise holds: `ResolveRound` fails only on `stat` (`internal/shedadapters/round.go:27-48`), and `refusal-spec.md:33` puts raw I/O failures on a transient row.
  - A `latestRound` error now refuses with the existing `circlingWayRetry` (`internal/loomcli/circling.go:43`), and a `hardCap` error still refuses with `circlingWayHardCap`.
  - "circling: hard cap unresolved" is scoped to loom.yaml or darn.yaml failing to load.
    "circling: decision record failed" widens its trigger to cover the unreadable latest round, and its way forward stays the same.
  - In `TestCirclingVerb_Refusals`, the "latest round unreadable" row now wants "re-run the verb".

## Disputed

None.

## Deferred

None.
