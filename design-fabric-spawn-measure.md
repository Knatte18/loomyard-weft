# Measure fabricengine tier 1 wall and git spawns before and after e19a2e86

`fabric-spawn-measure` · Task · research, fabric, git, gate · running · Drift-Recheck

## Seen (orch, 2026-11-09)

`fabric-serial-git-spawns` landed as `e19a2e86` ("fabricengine: read committed files and HEAD without a git process").
Its landing note says the tier 1 wall and the git process count before and after were not recorded,
so what it saved is unknown.
It names other frequent read-only git calls still spawned in the package, such as `diff --name-only` in `diffNames`.
The earlier shard measurement is `docs/research/speed/fabricengine-shards-2026-11-09/report.md`,
with its runner `docs/research/speed/fabricengine-shards-2026-11-09/shard-runner.sh`, which sets `GIT_TRACE2_EVENT`.

## This task

Measure, on this machine, `internal/fabricengine` tier 1 at two commits: `e19a2e86^` (before) and `e19a2e86` (after).
Run `lym gate test --tier 1 ./internal/fabricengine` with the shard runner's one-process form at each commit:
exactly one run per commit, two runs in all.
For each run, record the tier 1 wall, the total git process count,
and the count per git subcommand and per production call site in `internal/fabricengine`.

Write a short report beside the earlier one, under `docs/research/speed/`, holding:

- the wall and the git process count before and after, and the difference;
- the ten git subcommands and call sites started most often after the change, each marked read-only or writing,
  as the input for a next task that serves more read-only calls from Go.

## Limits

- No code change: the task lands the report only.
- Exactly two measured runs; a run that fails is reported as failed, not repeated.
