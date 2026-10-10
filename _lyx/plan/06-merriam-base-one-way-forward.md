# Card 6 — merriam-base-one-way-forward

**Edit:**
- `internal/websterengine#merriamBaseError`
- `internal/websterengine#Run`
- `internal/webstercli#websterCLI.executionBatches`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/merriambase_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/cli_test.go`

**Uses:**
- `internal/websterengine#MerriamBase`
- `internal/websterengine#autoRebaseline`
- `internal/websterengine#ErrAutoRebaseline`

**Intent:**
D5: `MerriamBase`'s error states the file and the cause and no way forward, so each caller words its own and a message holds exactly one `way forward:` clause.

`merriamBaseError` drops its trailing "; way forward: transient, re-run the verb", keeping "webster: compute Merriam's start base, reading <file>: <cause>".
`Run`'s fresh partition closure wraps the error as "<error>; way forward: transient, <re-entry step>" through `deps.reentryStep()`.
`websterCLI.executionBatches` and the Merriam-base call in `websterCLI.rebaselineCmd` each wrap it as "<error>; way forward: transient, re-run the verb".
`autoRebaseline` is unchanged: its transient wrapper already adds "way forward: transient, <re-entry step>" and `ErrAutoRebaseline`, so the Stuck mapping in the shed adapter holds through the existing row of `TestWebsterProducer_OtherEngineErrors`.

The "Merriam start base unreadable" row of `contracts/specs/refusal-spec.md` says the message names the file and the cause and that the verb adds the way forward; the "Webster row: auto-rebaseline transient" row already covers the row's own wording.

Tests:
`TestMerriamBase`'s absent-overview subtest asserts the message names `00-overview.md` and contains no "way forward".
`TestRun_AutoRebaseline`'s row "a failed read of Merriam's base" asserts, beside its existing checks, that the message holds exactly one "way forward:" substring, ends in the re-entry step, and never contains "re-run the verb".
`TestRun_RefusesBeforeSpawn` gains a row that plants a directory in place of `CLAUDE.md` before the first init, expecting a message naming `CLAUDE.md` with "way forward: transient, " followed by the run's re-entry step, `lyx webster run` when the fixture sets none, and no recorded state.
`TestRebaselineCmd_Refusals` gains a row planting a directory in place of `CLAUDE.md` in the worktree, expecting "way forward: transient, re-run the verb" once.
`TestValidateCmd_BatchesFlag` gains an assertion for the no-state partition through `executionBatches`: with a directory planted in place of `CLAUDE.md`, the `--batches` call refuses naming `CLAUDE.md` and "way forward: transient, re-run the verb".

**ImpactSummary:** the shared error loses its way forward, three of `MerriamBase`'s four callers gain their own, and `autoRebaseline`'s existing transient wrapper already supplies the fourth's; no signature changes.
