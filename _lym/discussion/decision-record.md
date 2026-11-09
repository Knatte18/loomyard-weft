## Goal

```markdown
## Seen (orch, 2026-11-09)

`docs/research/speed/fabricengine-shards-2026-11-09/report.md`: the package starts about 65 thousand git processes,
and the mean one took 6.8 ms as one process, 8.0 ms with two shards and 10.6 ms with four, so more concurrency slowed each spawn.
The serial phase is 59.5 percent of the wall, and a spawn removed from a serial test shortens it by the spawn's whole latency.
Hub copies are about 3 percent of the spawns.
On OSL-1046 each spawn also pays for Cortex XDR's scan, so the saving there is larger than the spawn time alone.

## This task

It runs after `fabric-serial-phase-tests` has made the longest serial tests parallel, and works on what is still serial then.

1. Count: run the top-level tests of `internal/fabricengine` that are still not marked parallel with `GIT_TRACE2_EVENT` set,
   and count the git processes each starts, per git subcommand and per production call site.
2. Replace: serve the most frequent read-only git calls from Go instead of a git process,
   as the `.git` worktree reader and the Go-side git config writer did before.
   Keep every call that writes or that Go cannot answer the same way git does.
3. Measure the package's tier 1 wall and its git process count before and after, on the same machine, and record both in the landing note.

## Limits

- No change to what any production path reads or decides: a Go read answers exactly what the git call it replaces answered.
- Nothing outside `internal/fabricengine`, `internal/gitkit` and their tests changes, except a caller of a function whose signature the task changes.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-09): Measurement runs allowed for this task

Decision: The writer may run lym gate test --wait --tier 1 ./internal/fabricengine with the shard-runner.sh exec for the task's count and before/after measurements. Cap: one counting run with GIT_TRACE2_EVENT, one before run and one after run; no other gate runs beyond Darn-Write's own verify gate. Record the numbers in the change description.

Rationale: The board entry asks for the count and the before/after measurement; a static pick without them cannot show the saving. A fixed cap bounds the time.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
