# Board model: kind, labels and intake

This session resumed with every batch already recorded `done`; it began and forked nothing for them.
`lyx webster status` confirmed all ten batches terminal `done`, with no pause and no open batch.

The ten batches, in plan order:

- 01 `board-label-vocabulary`: `board.yaml` gained the types and labels lists, read through a `Vocabulary`.
- 02 `board-alias-removal`: the hidden notes group and the promote-note verb were deleted.
- 03 `board-kind-and-labels`: kind and labels replaced tier and type across the model, render, listing and promote.
- 04 `board-load-migration`: old-shape `board.json` entries convert on load and persist on the next write.
- 05 `board-label-filter`: list and find take a repeatable `--label` filter.
- 08 `board-issue-records`: the `issues` field and the store-side import and fold.
- 06 `board-upsert-body-file`: upsert reads its body from `--body-file` or stdin.
- 07 `inbox-github-calls`: `selfreportengine` lists, fetches, comments on and closes inbox issues.
- 09 `board-intake-verbs`: the `lyx board intake` list, import and close verbs.
- 10 `board-model-docs`: package doc, overview, skill and sandbox docs, and the D13 label list.

No deviations from the declared file-ops were seen in this session, since no digest was read.

## Verify gate fixes

The gate's first attempt failed on `BenchmarkUpsert` at `n=10`, `n=100` and `n=1000`, each with `RunCLI upsert exit 1` at `bench_cli_test.go:66`.
The fixer fork reports the cause: the benchmark seeded the board at `<repo>/board`, but the board CLI reads `<hub>/_board`, so the upsert created a new entry instead of updating `task-0`.
Since card 3 a new entry defaults to a note and needs exactly one type label, so it exited 1.

The fork's commit `a58e3ef64`, `fix: seed the CLI board benchmarks in the hub's _board with the types and labels lists`, changed two files:

- `bench_cli_test.go`: `seedWikiRepo` moves the seeded board to `fabricengine.BoardDirName` next to the repo root.
- `bench_test.go`: the seed `board.yaml` carries `types: [bug, enhancement]` and `labels: [undecided]`.

The fork reports the three benchmarks and `go test -tags integration ./internal/boardengine/...` passing, with a clean worktree.
