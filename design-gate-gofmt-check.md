# No gate or test catches gofmt drift

`gate-gofmt-check` · Note · enhancement

## Seen

On 2026-10-10 `gofmt -l internal/loomcli/` on main (57243019b) lists `internal/loomcli/bootstrap_test.go` (a trailing blank line), and every test tier passed; the `webster-replan-state` conflict agent noticed it.
No test or gate step runs gofmt.

## Idea

Add a gofmt check (`gofmt -l` over the module must print nothing) to the test suite or the verify gate, and fix the file in the same change.