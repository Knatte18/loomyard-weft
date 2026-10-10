# Card 16 — start-keeps-status-strand

**Edit:**
- `internal/loomcli#loomCLI.startCmd`
- `internal/loomcli#mustUseLLMDriverArm`
- `internal/loomcli/smoke_driverstrand_test.go`
- `docs/overview.md`

**Delete:**
- `internal/loomcli#removeStatusStrands`
- `internal/loomcli#TestRemoveStatusStrands`

**Uses:**
- `internal/loomcli#loomCLI.ensureStatusStrand`

**Intent:** D11's last line: `lyx loom start` keeps the status strand on an llm-driven run instead of removing it, since the driver pane no longer shows the run's steps once the loop runs detached.
`startCmd`'s step 4 calls `ensureStatusStrand` on both arms, so the recorded driver selects the spawn alone; `mustUseLLMDriverArm`'s comment and the verb's `Long` text drop the strand branch.
`start.go`'s file header, which today says the status strand is "kept on a go-driven run, removed on an llm-driven one", says the status strand is ensured on every run.
`removeStatusStrands` loses its only caller and goes, with the test that drove it.
`smoke_driverstrand_test.go`'s three bootstraps assert exactly one status strand after each instead of none.
`docs/overview.md`'s `start` sentence in the loom module entry and its bootstrap bullet in the execution stack say the status strand is kept or added on every run.

**ImpactSummary:** One call site in `startCmd`, its file header and one test; the status strand now exists on llm-driven runs, which no Go reader branches on.
