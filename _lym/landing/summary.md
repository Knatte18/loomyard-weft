# Drift accept refuses unknown tests and keeps the output it accepts

Two interim repairs to Drift ahead of the Drift rework; nothing else of Drift changes.

## What changes

`lym drift accept <test>` now checks its test name.
The name must match a file under the run's baseline profile directory, with or without the file's last extension (a dotfile has none, so its whole name counts).
Any other name, including an empty one, is refused, naming the test and the directory.
Before this, a typo or a test that never drifted joined the compare's skip list and hid that test from every later compare.

The accepted profile now records each accepted test's output as it was when the test was accepted.
`lym drift accept` captures the current code's output first, in a gate slot and outside the session's run budget, and keeps the test's profile files.
Drift-Verify and Drift-Recheck build the accepted profile from the baseline with each accepted test's files replaced by the kept output, then compare HEAD against it.
Before this, Drift-Verify captured the profile at the end of its pass, so an agent that accepted test X and then edited code that moved X again left a profile recording the moved output.
Now that later move shows as drift, and Drift-Verify ends Stuck naming `lym drift accept` again, `lym drift withdraw` or a fix, then `lym loom resume`.
The Drift-Verify agent's instructions say so: accept a test only once its output is final.

## Design decisions

- `lym drift scope --skip` stays unchecked, which narrows the operator's wanted item 1.
  It runs at Drift-Baseline before any capture, and a capture writes no file for a skipped test, so a check against the baseline directory would refuse every partial scope.
  The capture harness already refuses an unknown skip name; the partial scope itself goes in the Drift rework.
- The accept verb runs `driftengine.CheckAccept` before it captures, so a refused acceptance costs no capture, and `driftengine.Accept` repeats the same checks before it keeps the output.
  The verb re-reads the run status after the capture, which can last minutes, so the acceptance is attributed against the status at the time it is recorded.
- Two accepts in one worktree would share the accepting directory, so the verb holds `Paths.AcceptLock` and refuses while another accept holds it.
  The engine functions take no part in that lock.
- New files and directories under `.lym/drift/`: `accepting` (capture target), `accepted-tests` (kept outputs), `accept.lock`.
  `Accept` replaces an earlier kept output only after the new files are moved, so a failed move loses no earlier output; `Withdraw` drops the kept output after the scope record is written.
- `RunAcceptedCapture` and the `CaptureAccepted` seam are replaced by `RunAcceptanceCapture`, which captures for one test, and by the `CompareAccepted` seam in Drift-Verify.
  The scope record's accepted-profile commit now means the commit a compare against the accepted profile last passed at, and Drift-Recheck's lapse reason says so.
- `docs/overview.md`, the `driftengine` and `driftcli` docs and the verify agent's seat text are updated to match.

## To check by hand

- `accept-name-match-admits-non-test` (class design, MEDIUM, in `internal/driftengine/accepted.go`, `testFiles`): the name match admits a name that is a file under the baseline directory but not a test, such as a non-profile file that happens to sit there.
  Decide whether the baseline directory holds only profile files.
- `accept-verb-orchestration-untested` (class design, MEDIUM, in `internal/driftcli/cli.go`, `acceptTest`): the verb's order of check, capture and record is tested only through `internal/driftcli/accept_integration_test.go`.
  Check that the test covers the order you rely on.
- Not run: an accept against a real capture command on a live Drift run.
