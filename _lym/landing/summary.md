# Run a gate key's report steps beside its last tier

A `lym gate test --gate <key>` call with `reports` ran the reach, redundancy and mutation steps one after another once every tier and lint had ended.
`docs/research/darn-slowdown-measure.md` measured that tail at about 75 to 105 s per verify gate, 4 to 7 % of a darn run.
The steps now judge in the background while the last tier runs, so their time overlaps the tier's instead of following it.

## What changes

- `internal/gatecli/run.go` starts the steps beside the first run of the plan's highest tier, tier2 for the hub's `darn-write` key, when no earlier run failed.
  They judge while the rest of the plan and the lints run.
  A run that fails after the start cancels them, and the call judges red-gate reports as before.
  A halted call cancels them and writes no report, as before.
  A call whose earlier run failed judges red-gate reports after its lints, as before.
- The reports are still written after the lints, and the call's exit code never depends on them.
  `internal/gatecli/reports.go` holds the new chain: each step prints into its own buffer while it judges,
  and the call prints those lines, then the report's own, in step order, so stderr reads as before.
- Each step's `run` splits into `judgeReport` and `writeReport`, so the background chain judges without writing.
  The step tests call the two in turn.
- The steps' memory guard samples every process under the call, which now includes the last tier's test binaries.
  It takes a `foreign` set of the running tier commands, recorded by `runInvocations`, and leaves a test binary below one to the tier's own guard.
  `TestMemoryGuard` gains a row for that.
- Each step still takes its own gate slot.
  It now asks while the last tier holds its slots, so on a pool too small for both it waits for the tier's release and gains nothing.
- The red-first gate is untouched.

The package doc of `internal/gatecli`, the `Reports` field doc in `internal/gateslot` and the gate template's comment now say the steps judge beside the last tier.

## Design decisions

- Concurrency was chosen over running the reports only on the last verify gate before a handoff:
  a gate cannot know it is the last, since the reach turn turns a pass into a writer turn.
- No before-and-after verify timing was made.
  The decision record's added entry drops the entry's bound to time one verify gate before and after the change:
  an after-timing needs a `lym` built from this branch, since every gate runs through the installed one,
  and the gain is to be measured from archived run records after landing.

## Merge danger

**Door:** two-way
**Blast radius:** gatecli
