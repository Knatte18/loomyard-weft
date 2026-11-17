# Report the trial of the coverage-map tooling

Adds `docs/research/speed/map-tooling-trial-2026-11-17/report.md`, linked from `docs/research/README.md`.
It records a run of `lym gate test --wait --map` and `lym gate redundant` on a small and a heavy package, before `redundancy-sweep-existing` relies on them.
No code changes.

## What the trial found

- `lym gate test --map` ran `internal/boardengine` (136 tests) in 12.4 s cold and 5.0 s warm, with a tier0 bound of 4 side-by-side runs and no memory cap.
- Map entries matched single-test `-coverprofile` runs on every test checked, five in all, by covered blocks per file.
- A second map build writes the same sets but not the same bytes: the `funcs` order of four `boardengine` tests differs.
- Neither verb left a changed file in the worktree or wrote a gate run record.
- `lym gate redundant internal/boardengine` took 21 min 39 s and 2.1 GB, and listed 14 candidates, each at 0 seconds.
  Mutants ended by timeout or memory cap count as kills; the trial did not test whether the candidates are deletable.

## Design decisions

- `internal/lymdirs`, named in the brief, has no tier0 or tier1 test and `--map` builds nothing for it, so `internal/durfmt` stands in as the small package.
  `durfmt` has two tests, so the cross-check covers two there, not three.
- The cross-check uses `lym gate test ... -- -run <test> -coverprofile=<path>`, since a raw `go test` is refused.
  Its five run records are the only ones the worktree's slug left in the gate's run directory.
- The `/tmp/lym-1000` scratch root is shared with other sessions, so leftovers there could not be attributed; the report says so.
- Three notes filed instead of fixes: `map-entry-order-unstable`, `map-test-seconds-zero`, `map-single-test-crosscheck`.
