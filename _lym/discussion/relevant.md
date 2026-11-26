# Relevant files

- `internal/reportengine/archivedrecords.go` — decodes older archived parent-review rounds; the new ask copies must not collide with its directory
- `internal/reportcli/cli.go` — the `report` cobra group the new verb joins
- `internal/burlerengine/parentreview.go` `ParentReviewFilesOf`, `RecordParentAsk`, `AnswerParentAskOrUnreached`, `ExpireParentAsk` — the live parent-review ask record and its reports under `.lym`
- `internal/loomcli/prfindings.go` — `lym loom pr-findings`, which stores findings only under `.lym/loom/`
- `internal/shedrun/usage.go` `LandedSizeRecord` — Finalize-time record in the run's records, precedent for the index row
- `internal/shedrun/paths.go` — run scratch and pull-request record paths
- `internal/fabricengine/junctionnames.go` — `_lym` as the committed directory set
- `internal/driftengine/paths.go` — drift's tracked and scratch directories
- `internal/loomengine/config.go` `LoomReviewsDir`, `LoomScratchDir` — tracked reviews root and loom scratch
