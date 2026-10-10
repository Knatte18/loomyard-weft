# Webster round 2 — fixer report

All three findings were validated against the code and fixed, one commit each.

## Fixed

- **F2** (`ba5cc4b01`) — `prebuildLyx` in `cmd/testtiming/prebuild.go` now takes a gate slot through `takeGateSlot` for the build alone and always passes the slot's `-p` (the template's `GoParallel` outside a hub); the returned environment carries none of the slot's variables.
  The `parallel` parameter and its zero arm are gone.
  The review cited only `main.go:174`, but `redundancy.go:350` made the same uncapped call and is fixed by the same change.
  `-resources` now builds first and takes its measuring slot afterwards, so the build never nests a second slot inside the one it holds.
- **F1** (`6dcc3ea77`) — `prebuildLyx` takes the `go` binary it runs, and the callers pass `"go"`.
  `TestPrebuildLyx` gains the row "a tagged run outside a hub caps its build at the gate template's -p".
  That row runs the test binary as a recording stand-in `go`, through a `standInGoRecordEnv` role in `testmain_test.go`, and asserts the exact arguments `build -C <root> -p <template GoParallel> -o <bin> ./cmd/lyx`.
  The finding is a coverage gap, so a row was added, not a new top-level test.
  Since F2 the build always passes `-p`, so no zero arm remains to cover.
- **F3** (`fb8c50c11`) — added one-line file headers to `internal/dotgit/dotgit.go`, `internal/testkit/tmuxkit/allow_none.go` and `internal/testkit/tmuxkit/allow_tmux.go`.

Verified with `lyx gate test ./cmd/testtiming ./cmd/lyx`, `lyx gate test --tags integration ./cmd/testtiming` (with `-race` on `TestPrebuildLyx`), `lyx gate test ./internal/dotgit ./internal/testkit/tmuxkit ./cmd/lyx` and `lyx loom lint-comments`, all green.

## Disputed

None.

## Deferred

None.
