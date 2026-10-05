# Round 1 fixer report

All three findings were NITs in `internal/configcli/configcli.go`, and all three are fixed.

- F1: the `menuCmd` comment now breaks before "so its RunE is written by hand."
- F2: the `buildConfigLong` editor sentence is reflowed, so the short ragged line is gone.
- F3: the new `baseDirOf` helper replaces the three spellings of the baseDir join.

The three fixes share one commit instead of one each, because they touch the same file in one pass.
`gofmt`, `go vet` and `go test ./internal/configcli` pass.

## Deferred

Nothing.
