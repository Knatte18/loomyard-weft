# Fewer git processes in the serial phase of internal/fabricengine tier 1

`fabric-serial-git-spawns` · Task · enhancement, fabric, git · draft

## Seen (orch, 2026-11-09)

`docs/research/speed/fabricengine-shards-2026-11-09/report.md`: the package starts about 65 thousand git processes, and the mean one took 6.8 ms as one process, 8.0 ms with two shards and 10.6 ms with four, so more concurrency slowed each spawn.
Hub copies are about 3 percent of the spawns.

## Idea

Count git spawns per test in the serial phase and replace the ones a Go-side read can serve, as `dotgit-worktree-reader` and `git-config-from-go` did.
On OSL-1046 each spawn also pays for Cortex XDR's scan, so the saving there is larger than the spawn time alone.
