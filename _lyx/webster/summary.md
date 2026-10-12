# run-robustness: tolerant run records and the gate CPU watchdog

All three batches recorded `done`; the first plan-level verify gate failed on one test, and a fixer fork fixed it.

## 01-board-encoding-pin (head `6f6270e05`)

Board.json's Save bytes are pinned, and state reads run records with unknown fields accepted.
Every state write keeps the old file's unknown keys, `ReadJSONStrict` is renamed `ReadJSONNoMkdir` across the module, and `seed.json` decodes tolerantly.
The rename touched many call sites outside the cards' own targets, in `battencli`, `battenrecipe`, `loomcli`, `loomrecipe`, `loomshed`, `pairteardown`, `shedcli`, `shedengine` and `shedverbs`; they are the reported deviations.
The fork reported one scripted edit, a shell heredoc append to `internal/state/state_test.go`, so an edit audit may flag it.

## 06-batten-cleanup-tolerant (head `f7f699e6a`)

Batten cleanup judges a status with an unknown key.
`PATTERN-tolerant-records`, its background file and its scan landed.
`busy_quit_sec` joined the hub-wide gate config and `Limits`, `proc` samples a process tree's CPU time and walks ancestry on Linux, and `gateslot` renders and finds the watchdog's quit line and names its way forward.
No deviations.

## 11-gate-test-watchdog (head `51adfecb1`)

`lyx gate test` SIGQUITs a CPU-busy Go test binary.
Webster's verify gate skips the rerun after a hang and names it, and the loom and landing verify gates name a watchdog quit and `busy_quit_sec`.
No deviations.

## Verify gate fixes

The gate's first attempt failed on `TestLoadState_UnusualFiles/legacy_integration_records_still_load` in `internal/websterengine/state_test.go`.
It asserted the saved `state.json` drops the retired `assertedModel` key, but batch 01 made every state write keep unknown keys.
The fixer flipped the assertion to expect that `assertedModel: "opus"` and `integrationFix` survive a save, and updated the test's doc comment.
Its reply says `lyx gate test --tags integration ./internal/websterengine` passes and the worktree is clean.
The fix is commit `067b74ec9`.
