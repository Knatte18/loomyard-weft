# Round 1 fixer-report

## Changes

- F1 (fixed as proposed): `addStrand` in `internal/creelcli/smoke_helpers_test.go` no longer passes `--anchor own-window`, and its doc comment no longer cites issue 673.
  `internal/reedcli/add.go` defaults `--anchor` to `own-window`, so the flag was redundant.
  Verified with `go vet -tags tmux ./internal/creelcli/`; the file is behind the `tmux || llm` tags, so the smoke tests themselves were not run.

## Disputed

None.

## Deferred

None.
