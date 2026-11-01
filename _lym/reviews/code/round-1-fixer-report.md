# Fixer-report, code review round 1

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  `composeButtonCommand` and `composeConfirmedButtonLine` in `internal/reedengine/panebin.go` wrap the directory change in `sh.DiscardOutput`, so a gone run directory writes nothing to stderr and tmux opens no view mode.
  Their doc comments say so.
  Tests: the click-line test in `panebin_test.go` and the confirmed-line rows in `buttonclick_test.go` pin the discarded directory change for both dialects; `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` in `bindings_integration_test.go` gains a step that removes the run directory, clicks, and asserts `pane_in_mode` is 0.
  Commit `f61c0e09`.
- **F2 (LOW), fixed as proposed in part.**
  The pwsh click-line row is added beside the POSIX one in `panebin_test.go` (same commit).
  The proposed `ExitSuccess` assertion in `shell_test.go` is not added: `TestShell_Exit` already asserts `ExitSuccess` for both dialects.
- **Self-check LOW (consistency), fixed.**
  `internal/reedengine/doc.go` described only the verb's output as discarded in two places; both now name the directory change too.

## Verification

`lym gate test ./internal/reedengine` passes at tier 0.
The new real-tmux step is unrun: tmux-tier runs are refused in this environment, so it rests on reading the code and the review's tmux 3.6 probe.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, stale `doc.go` sentences on the discard. Fixed as proposed.
- fix-proposal: no findings against the proposals; it repeated the `doc.go` LOW above (merged).
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
