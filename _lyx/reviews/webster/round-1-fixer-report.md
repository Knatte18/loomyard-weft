# Webster round 1 — fixer report

Every finding's premise held against the code and the plan, and all four are fixed, one commit each.
The plan's `## verify:` passes in full after the last fix: build, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...`.
`lyx loom lint-comments` passes.

## Fixed

- **F1** (`8f42fdfb6`): `TestBurlerProducer_Call_WritesRoundUsageRecord` and `TestWriteRoundFacts_LensSection` call `t.Parallel`, as do their subtests, which share no state.
  `TestBurlerProducer_Call_ClusterExclusions` and `TestSessionUsage_SumsParentAndForks` stay serial, and their doc comments now name the process-global logger and `HOME`.
  `go test -race` passes on both packages.
- **F2** (`08c53ee3a`): `validateFanKeys` returns at once when `discussion_fan` and `plan_fan` are both empty, so `burler.yaml` is read only when a fan key is set, and an unset fan loads exactly as before.
- **F3** (`f21522b18`): card 5 names `transcriptTokens` as a Create member, so it is kept rather than deleted.
  `sumTranscriptTokens` is folded into it: `transcriptTokens(path, includeSidechain, skipIDs)` is now the one transcript summer, `SessionUsage` calls it for the parent and each fork, and its doc comment stands on its own.
- **F4** (`711124aca`): `gofmt -w` on `internal/loomengine/config.go`, `internal/loomengine/config_test.go` and `internal/shuttleengine/claudeengine/usage_test.go`, all clean at the merge base.

## Disputed

None.

## Deferred

None.
