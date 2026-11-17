# Trial the coverage-map tooling before the redundancy sweep

`map-tooling-trial` · Task · research, gate, testkit · starting · Open-Window

## Seen (orch, 2026-11-17)

`gate-test-map-mode` (`b92c95ee`) replaced `lym gate testmap` with `lym gate test --map`,
which runs each test in its own process, in parallel, and writes each package's map whole.
`lym gate redundant` reads those maps and confirms candidates by mutation.
Neither has run on a real package since the change, and the survey `redundancy-confirmed-survey` (`53d40bd0`) found that
`lym gate redundant` leaves mutant files in the source tree and that 27 of 30 sampled confirmations were false.
The operator wants the tooling proven before `redundancy-sweep-existing` uses it (operator, 2026-11-17).

## Idea

A trial of the map tooling, run in the task's own worktree, with its report landed as a commit:

- `lym gate test --map` on two packages, one small and one heavy (`internal/boardengine` or `internal/seatengine`):
  wall time, peak memory and the parallelism it used against the tier's `parallel` and `memory_mb`;
  that each test's map entry matches what a single-test `go test -cover` run of that test records, checked on three tests per package;
  that a second run writes the same map;
  that the run writes no green run record and leaves nothing behind in the worktree or the scratch root.
- `lym gate redundant` on the same two packages:
  its candidate list and wall time, and `git status` after it, which must show no mutant or other changed file.
- Each defect found is filed as its own note; the trial fixes nothing.

## Cap

- Two packages, each mapped twice; one `lym gate redundant` run per package.
- No model or effort choice: the runs are lym commands, and the report is written by the darn writer at its default model and effort.
