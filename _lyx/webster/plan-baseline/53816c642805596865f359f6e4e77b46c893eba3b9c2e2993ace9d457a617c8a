# Card 12 — fabriccli-dedupe

**Edit:**
- `fabriccli/cli_test.go`
- `fabriccli/envelopecontract_integration_test.go`
- `fabriccli/merge_cli_integration_test.go`

**Create:**
- `fabriccli/help_test.go`

**Uses:**
- `internal/fabricengine#ErrNoWarpBinding`
- `internal/fabricengine#ErrWarpBindingMismatch`

**Intent:**
Give each fabriccli behaviour one owning test (D7, #325 §5).

Remedy strings: `cli_test.go`'s pins of `"has no recorded warp binding"` and `"lyx fabric clone <weft-url> <warp-url>"` go; the CLI emits `CloneHub`'s one-argument refusal, whose single owner card 11 made `TestCloneHub_UnboundWeftNamesTwoArgForm` in `fabricengine/warpbinding_clone_integration_test.go`.
In their place one test asserts, generically, that an engine refusal reaches the envelope unchanged: drive `clone <weft-url>` into the no-binding refusal and require the envelope's error to equal the engine error's `Error()` text exactly, not a substring.
What this lets through: a CLI-side change to remedy wording now fails only that equality check, which is what bounds it.

Help prose: `TestRunCLI_CommitHelp`, `TestRunCLI_PullHelp`, `TestRunCLI_PullShortNonEmpty` and `TestRunCLI_MergeInHelp` lose their prose pins.
Their behavioural parts become `cmd.Flags().Lookup(...)` checks on the built command tree in the new untagged `help_test.go`, which spawns nothing; the prose-only tests are deleted.

Conflict envelope: the shape is asserted by the unit `TestErrConflictsWithRecord_ConflictEnvelopeShape` (`envelope_test.go`), `TestRunCLI_MergeConflictEnvelopeKeepsPartialFalse`, and the first half of `TestRunCLI_MergeInConflictThenContinueConcludes`.
Keep the unit test and `TestRunCLI_MergeInConflictThenContinueConcludes`; delete `TestRunCLI_MergeConflictEnvelopeKeepsPartialFalse` after moving into the kept integration test any assertion it makes that neither kept test does (D9's exception: the behaviour stays asserted in its named owner).

List every deleted leaf test in the commit body for the test-count accounting.
Apply D10 to every touched test.

**ImpactSummary:** fabriccli tests only: duplicate remedy, help-prose and conflict-envelope assertions consolidate onto one owner each; no production change.
