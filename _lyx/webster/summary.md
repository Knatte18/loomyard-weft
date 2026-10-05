# test-suite-measure: webster run summary

This session resumed a run whose progress trail already listed every batch as `done`, and `lyx webster status` confirmed it: all 21 batches are terminal `done` with a digest, and no batch was left to begin, fork or recover.
No `deviations` field was read, so this summary reports no deviations from the plan's declared file-ops.

The plan's work, as the card list records it:

- Measurement: `testtiming` reports per-package test count and serial time and takes `-tags` (01); the dated before-state with its slowest-tests table (02) and the after-speed timing on the same pre-retag test set (09).
- Speed seams and parallelism: per-instance timing seams for the watchdog daemon (03), the watchdog's reap timeout (04) and the verifyrun and verifytree waits (05); fabricengine's serial integration tests run in parallel (06); the weft/warp scan is prefiltered (07); gitoracle's tests run under the hermetic git environment (08).
- Tier split: PATTERN's Testing entries merged into five (10); the `llmkit` kit with its bounded `os/exec` exemption (11); the tier guards learn the `tmux` and `llm` tags (12); `smoke` retired onto `tmux` or `llm` (13); tmux-starting `integration` files moved to `tmux` (14); a guard keeps LLM spawns in the `llm` tier (15); the four tiers documented in docs, spec and stencils (16).
- Socket leak: tmuxkit removes the socket files of the servers it kills (17); ad hoc tmux keys in tests move onto tmuxkit (18); reed removes the socket file of a server it tore down (19).
- Redundancy: `testtiming`'s per-test coverage redundancy mode (20); the committed redundancy report and the post-retag timing (21).

## Verify gate fixes

The gate's first attempt failed on `reedcli.TestSmokeDotFillCrossClientControl`: pane `%1` never showed `DOTFILL-MARKER-BETA` within 20s.
The fixer fork reports the test already failed the same way at the pre-plan base commit: the toucher pane is about 8 rows, too short to render the second strand, so only `DOTFILL-MARKER-ALPHA` appears.
It changed the test to wait on `DOTFILL-MARKER-ALPHA` for the toucher attach, with a comment saying why, leaving the attach-complete check and the dot-run assertions unchanged.
The fix is commit `cddbf1e7a` (`fix: cross-client dot-fill control waits on the strand an 8-row pane can render`), touching only `internal/reedcli/smoke_dotfill_test.go`.
The fork reports `go test -tags tmux ./internal/reedcli` and `go test -tags integration ./internal/reedcli` pass; it did not run the plan-level verify, which the gate runs next.
