---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/driftcli/capture_integration_test.go:152
    summary: the existing capture-failure subtest asserts the verb succeeds for a capture that fails, which the new startMainCapture failure branch makes timing-dependent, and the verb's non-zero exit and the child's error envelope have no test
    fix: >-
      In TestCapture_OutlivesCaller, rewrite the subtest "a capture with the marker absent records failed" so it no longer requires env.OK from lymRun for a capture whose command exits non-zero at once.
      Accept both outcomes the new code allows: either env.OK, or a not-OK envelope whose error text contains the capture's reason (starts with "drift: the baseline capture ended failed", names "it ran in" and the log path) and the Drift-Await ways forward.
      Keep the waitFor on captureState == CaptureFailed.
      Add a deterministic row for the child: after the capture record reads failed, read the capture-child log named by the record's sibling capture-child-*.log under the drift logs dir and assert its last JSON line has ok false and an error holding the same reason.
      The verb's failure branch cannot be forced deterministically while the verb stays detached, so cover it through the two-outcome assertion above and keep the child-log row as the deterministic pin of the failure text and the non-zero exit.
  - id: F2
    severity: MEDIUM
    class: decision
    origin: lens:code-correctness
    location: internal/driftcli/cli.go:349
    summary: lym drift capture exits 0 for a capture that later exits 0 with no baseline file, because the verb reads the record right after the child's start and the record still reads running, so the Goal's first Part 1 item holds only for a capture that ended before the read
    fix: >-
      Leave the code in place and defer this finding to the run's parent.
      The Goal wants lym drift capture to fail non-zero with a stated cause when it produced no baseline file; the change description keeps the verb detached and records only the child, the record, lym drift status and the Drift-Await halt as carrying the failure.
      The parent records with lym loom decision add either that the detached verb plus a non-zero child exit meets the item, or that the verb must wait for the record to leave running, bounded by the capture step's timeout, and return driftengine.FailedCaptureError on failed.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/steprun.go:164
    summary: commandReport reads the whole log into memory for three lines and boundOneLine cuts the quoted tail from its end, so a long tail drops the final line, which is the line that usually states the cause
    fix: >-
      In commandReport, read only the log's last chunk (open the file, Seek to max(0, size-65536), read to the end, and drop the first partial line when the seek was not to 0) instead of os.ReadFile of the whole log.
      Cut an over-long joined tail from its start, keeping its last outputTailMaxRunes-1 runes behind a leading ellipsis, by giving boundOneLine a keep-the-end variant used only by commandReport; readChildReport keeps cutting from the end.
      Change the capture_test.go row "an over-long last line is cut to the output bound with an ellipsis" to expect the ellipsis first and the last outputTailMaxRunes-1 runes after it, and add a row whose last line is short after two long lines and must appear whole.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/orchcli/reconcilenotifier.go:19
    summary: nothing tests that InstallReconcileNotifier mails the hub's prime orch signed RECONCILE, that it reports another hub's notice not posted, that it installs nothing outside a hub, or that the root pre-run installs it
    fix: >-
      Beside the existing push notifier tests in internal/orchcli (notifierwiring_test.go and pushnotifier_integration_test.go), add rows for InstallReconcileNotifier in the same table or fixture those use.
      Install it from a hub's cwd and call the notifier hubreconcile holds with the hub's board dir and a line, then assert the prime orch's creel inbox holds that line from RECONCILE.
      Call it with another hub's board dir and assert it returns false with no error and posts nothing.
      Install it from a cwd outside a hub and assert no notifier changes.
      Extend the wiring test that checks the push notifier is installed by the root pre-run to check the reconcile notifier too.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/dirbase.go:357
    summary: a dir written as null or tilde is read as the literal directory null or tilde, so RebaseDirsOntoRoot flags it as naming no directory and the orch gets a false notice for what the config loader reads as unset
    fix: >-
      In RebaseDirsOntoRoot, skip a dir node whose Tag is "!!null" in the same condition that skips an empty or absolute value (dir == nil || dir.Tag == "!!null" || dir.Value == "" || filepath.IsAbs(dir.Value)).
      Add a row to TestRebaseDirsOntoRoot with the input "capture:\n  dir: null\ncompare:\n  dir: ~\n" that wants the bytes unchanged and no flagged line.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/driftengine/steprun.go:166
    summary: the unreadable-log clause of commandReport, the directory-and-output clause on a main capture timeout, an accepted capture timeout and a compare timeout, and the rule that OnEnd is not called when the command never ran have no test row
    fix: >-
      In capture_test.go add to the outcome rows of TestRunCapture a timeout row with wantEnds "; it ran in {dir} and printed nothing" (or its output), add the same suffix to the accepted-capture timeout row's wantReasonIn, and add the suffix to the compare timeout assertion in compare_test.go.
      Add a TestRunCapture subtest where the step's dir does not exist so runStep refuses before the spawn, with f.req.OnEnd set to a function that fails the test, asserting RunCapture returns the error and the record reads failed.
      Cover the unreadable-log clause through commandReport's own row only if the log can be removed between the command's end and the report in a fixture; otherwise drop that branch's message detail and keep one row on the wording "its log could not be read".
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/hubreconcile/reinterpret.go:647
    summary: notifyReinterpretations paths for no installed notifier, a notifier error and a mail not posted, and a Rewrite error that fails the walk, have no test row
    fix: >-
      In reinterpret_integration_test.go add to the table of TestEnsure_ReinterpretationMarkerIsCommittedAndMailsOnlyAFlaggedValue rows whose recordNotices replacement returns (false, nil) and (false, someError), asserting Ensure returns nil, the marker is committed and the file keeps its flagged dir.
      Add a row with no notifier installed (install one that restores nil, or run before any install) asserting the same.
      Add a row whose board drift.yaml is unparseable (for example "capture: {dir: [\n") on an anchored hub asserting Ensure fails naming drift.yaml, the tree is as before and no reinterpretation marker is left.
  - id: F8
    severity: NIT
    class: consistency
    origin: handler
    location: internal/configengine/config.go:143
    summary: the doc comment of ReinterpretedMarkerRel is written in terms of another named symbol, ConfigFileRel, which the code-quality self-contained rule forbids
    fix: >-
      Reword the comment to stand alone, for example "ReinterpretedMarkerRel returns the marker of the reinterpretation named id as a path relative to the board dir, the form a board commit lists."
---

### [BLOCKING] A capture-failure subtest the diff makes timing-dependent, and no test for the verb's non-zero exit

**Location:** internal/driftcli/capture_integration_test.go:152
**Issue:** `TestCapture_OutlivesCaller` runs `lym drift capture` with the marker absent, so the command exits non-zero at once, and requires `env.OK` at lines 153 to 156.
`startMainCapture` (`internal/driftcli/cli.go:349-356`) now reads the capture record after the child's start and returns `FailedCaptureError` when it already reads failed with the child's pid.
Whether the record still reads `running` or already reads `failed` when the verb reads it is a race, so the subtest passes or fails by timing.
No test covers the verb's new failure branch, the new `state` field of its result, or the child's `OnEnd` hook that writes the error envelope and exits non-zero (`cli.go:431-435`).
The Goal's first Part 1 item is that exit and its stated cause.

### [MEDIUM] `lym drift capture` exits 0 for a capture that then writes no baseline

**Location:** internal/driftcli/cli.go:349
**Issue:** `StartCapture` returns once the record carries the child's pid, and the child writes that record as `running` before the command starts (`capture.go:84-87`), so the new check at `cli.go:353` sees `failed` only for a capture that already ended.
A capture that exits 0 with no file after a slot wait and a spawn still reads `running`, and the verb exits 0.
The Goal wants `lym drift capture` to fail non-zero with a stated cause when it produced no baseline file.
The change description keeps the verb detached as a design decision and offers the child's exit, the record, `lym drift status` and the Drift-Await halt as carrying a later failure.
That departs from the Goal's literal wording without a recorded decision; the change description does not name it an unrecorded parent correction, so it is filed as a `decision` for the parent.

### [MEDIUM] `commandReport` reads the whole log and cuts the tail from its end

**Location:** internal/driftengine/steprun.go:164
**Issue:** `os.ReadFile` loads the whole command log to take its last three non-empty lines; a verbose regression harness can write a log of hundreds of megabytes.
`boundOneLine` cuts the joined tail to its first `outputTailMaxRunes-1` runes, so when the three lines exceed 400 runes the last line is the one lost.
The last line is the one that usually states the cause, as in the Goal's `The provided file path does not exist` case.
The test row "an over-long last line is cut to the output bound with an ellipsis" pins the cut from the end.

### [MEDIUM] The reconcile notifier that tells the hub's orch has no test

**Location:** internal/orchcli/reconcilenotifier.go:19
**Issue:** The second wanted item of Part 2 is that a hub's orch is told when a reconcile changes or flags one of its files.
`hubreconcile`'s tests record the notifier's calls, but the `orchcli` adapter that resolves the hub, matches the board dir, picks a drivable sender and posts through `NotifyPrime` with the program name `RECONCILE` has no test, nor does the root pre-run line in `cmd/lym/main.go` that installs it.
`notifierwiring_test.go` and `pushnotifier_integration_test.go` pin the push notifier only.

### [LOW] A null dir is flagged as a missing directory

**Location:** internal/driftengine/dirbase.go:357
**Issue:** `dir: null` and `dir: ~` parse to a scalar node with Value `null` or `~` and tag `!!null`, which the config loader reads as unset.
`RebaseDirsOntoRoot` tests only `dir.Value == ""`, finds no directory of that name, and flags the value, so the orch is mailed a false notice.

### [LOW] Failure-text and hook branches with no test row

**Location:** internal/driftengine/steprun.go:166
**Issue:** The clause naming the directory and the output is pinned for a non-zero exit and an exit 0 with no file, but not for a timeout of the main capture, the accepted capture or the compare, whose rows check the `timed out after ` prefix only.
The branch of `commandReport` for an unreadable log has no row.
The `RunCapture` doc says `OnEnd` is called once the command ran, and no row shows it is not called when `runStep` refuses a missing dir or `failCapture` returns early.

### [LOW] Delivery-failure and rewrite-failure paths of the reinterpretation walk have no test row

**Location:** internal/hubreconcile/reinterpret.go:647
**Issue:** `notifyReinterpretations` has three non-silent paths: no notifier installed, a notifier error and a mail not posted.
Only the silence on no lines is pinned.
A `Rewrite` error that fails the walk through `configsync.FileError` (`reinterpret.go:622-625`) has no row either; the generic failed-walk test may not reach it with a drift file.

### [NIT] A doc comment written in terms of another symbol

**Location:** internal/configengine/config.go:143
**Issue:** `ReinterpretedMarkerRel` is documented as "the shape ConfigFileRel returns", which makes it depend on a named sibling symbol.
The code-quality rule requires a doc comment to be correct to a reader who knows no other symbol.

## Rejected

- A reinterpretation flagged but not delivered (no orch recorded, mail failed) is lost, since the marker is written regardless: the change description records this as a design decision following `NotifyPrime`, and a flagged dir still fails by name before the spawn, so the operator gets a durable trace at the next run.
- A `ReadCapture` error after a successful start reports a failure for a running capture: `awaitRunningRecord` has just read the same record, so the second read failing is not a reachable case worth a branch.
- The anchor-wins rule rewrites a hand-fixed dir that also exists under the anchor: the change description states the rule, "a dir that exists under the anchor is taken as written for the old base, even when it also exists under the root", and the Goal asks for rewrite or flag of exactly the ambiguous case.
- The third Part 1 item, finding why the capture wrote no file and why the first re-capture reproduced it, is answered by the change description (all six harness cases skipped under scope `partial`, re-capture under the same scope) and needs no code; this is a valid answer, not a defect.
- Scope against the Goal: the `cmd/lym/main.go` line, the `PATTERN.md` entry `PATTERN-config-reinterpretation` and the `hubLocation` extraction are each licensed by Part 2's two wanted items; the `PATTERN.md` text matches the entry in the constraints section.
- A missing `Pattern-Departure:` trailer: the range carries none and none is needed.
- `TestFailedCaptureError` sitting in a tier 1 file for a pure function: the file's other tests spawn commands, and a file names one tier.
- A non-mapping or empty `drift.yaml` returning unchanged has no row (a lens LOW): the branch returns the input untouched and the neighbouring rows pin the same return, so it is not worth a row.
