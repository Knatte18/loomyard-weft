# Webster-Review round 2 — fixer report

All four findings were validated against the code and fixed, one commit each; `go test ./...` and `lyx loom lint-comments` pass.

## Fixed

- **F1** (`e17396e09`): `fakeGit` gains a `dirtyPathsErr` field, and `TestRecoverSpawnOrAttach_StartFailures` becomes a table with a second row in which git status fails during `recoverSpawn`.
  The row asserts the error wraps the status failure and ends in `way forward: transient, re-run \`lyx webster recover-batch 1\``, that no strand was prepared and no batch state recorded, and that a re-run spawns once the failure clears.
  The not-ready-start row keeps its prior assertions.
  The comment gate then found the test's doc comment broken mid-sentence; `5de8b2969` rejoins it to one sentence per line.
- **F2** (`288869670`): `Run` parses the plan through `planparser.ParsePlanFrom` with a read that keeps the overview's bytes, and hashes the frame from those bytes through the new `overviewFrameHashOf`.
  The second read is gone, so no refusal remains there.
  A frame failure on bytes the parse accepted is a violated invariant and panics.
  `run` is dropped from the "overview frame unreadable" spec row.
- **F3** (`29eb74975`): rebaseline's index-only read refusal now ends `transient, re-run \`lyx webster rebaseline\` with the same \`--card\` flags`.
  The spec row and `TestRebaseline_OverviewFrameUnreadable`'s expected suffix match it.
- **F4** (`b3460affd`): `TestMigrateConfig` gains the `master_base`, `custom`, `startup_context` row, expecting `orientation` where `master_base` stood.
  A last-drop-index regression would put it after `custom` and fail this row.

## Disputed

None.

## Deferred

None.
