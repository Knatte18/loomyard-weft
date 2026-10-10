# Webster round 2 — fixer report

## Fixed

F1 and F2 share one root cause and one fix, commit `d2ad9fc55`.

- **Cause:** card 6's `currentProducer` (`internal/loomcli/start.go`) read the status file with `state.ReadJSONStrict` before every spawn, and `runDriverSpawnAndWait` refused on any error.
  A status file with an unknown field (F1) or truncated JSON (F2) was refused at bootstrap, so no driver ran and no `driver.log` was written.
- **Fix:** `currentProducer` treats a `state.ErrDecode` failure as no producer, logs it at warn and returns `""`.
  Over a parked merge, the ordinary `merge_in_progress` refusal stands, because an unknown producer is never a landing row.
  Otherwise the spawn proceeds and the driver's own read gate reports the decode failure.
  A read or lock failure still refuses.
  The card's `refuseOverUnfinishedMerge` signature is unchanged.
- **Tests (coverage gap):** `TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds` gains two rows: a clean pair whose status file has an unknown field, and one whose status file is truncated.
  Both assert the probe ran and the starter was reached.
  `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` gains a row: the own leftover under an undecodable status file keeps the `merge_in_progress` refusal without reading the parent branch.
  All three rows failed before the fix with the refusal the publish log shows, and pass after it.
- **Verification:** `lyx gate test ./internal/loomcli` and `lyx gate test --tags integration ./internal/loomcli` are green, and `lyx loom lint-comments` is clean.
  In the verify gate's tmux run, `TestSmokeBootstrapLifecycle/malformed_status_proceeds_to_the_success_envelope_and_logs_why` now passes.

## Disputed

None.

## Deferred

- **Verify gate: `TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why` now fails earlier, at `smoke_test.go:601`.**
  The failure is in the step's run-lock probe, before its `loom start` runs: `probe run lock: try acquire write lock: open .../loom-smoke-task/.lyx/shed/loom-smoke-task/run.lock: no such file or directory`.
  It failed the same way on all three verify runs after the fix, so it is deterministic, not a flake.
  On the publish-failure run, before the fix, this probe passed and the step failed later, at line 611.
  The malformed step's identical probe passes a few milliseconds after this one fails.
  The fix only changes the decode-error branch, which the steps before this one never reach.
  I found no production code on the branch that removes `.lyx/shed/<run>`.
  The smoke test is tmux-tier, and this session refuses tmux-tier runs, so I could not reproduce or bisect it.
  `ly:orch` chose to leave `smoke_test.go` unchanged rather than make the probe create the missing directory, since a vanishing run directory may be a real bug the fixture would hide.
  `ly:orch` is investigating it as a separate stop, starting from merge `6235b4043`'s bolt-and-store and daemon-wakeups changes.
