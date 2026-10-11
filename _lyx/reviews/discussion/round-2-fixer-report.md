# Fixer report — discussion, round 2

Every finding's premise was checked against the code and holds; all five are fixed in `_lyx/discussion/decision-record.md`, with the matching rejected alternatives and ledger entries in `_lyx/discussion/support-log.md`.

## Changes

- **F1 (MEDIUM, design) — fixed as proposed (first option).**
  Premise confirmed: `resumeParkedDriver` builds a fresh `reportPath` from `time.Now` and `newDriverReportRand()` on every call, and `driverResumeLine` embeds it.
  D8 now fixes the resume line's report path once per park: the first attempt stores it in the run's scratch directory beside the park marker (path declared in `internal/shedrun`), every rerun while the marker stays reuses it, and it is removed with the marker, so a rerun types the same line and D5's text-keyed record matches.
  Scope's loom-start bullet and the D8 acceptance criterion carry it.
  Keying the record per strand regardless of text is listed as a rejected alternative: a second, different notice would be dropped as landed.
- **F2 (MEDIUM, design) — fixed as proposed (first option).**
  Premise confirmed: `noticeSender.deliver` in `internal/seatengine/notice.go` retries `ErrSubmissionNotLanded`, and `deliverNotice` leaves the notice file on any send error.
  D5 now writes the pending-send record for both outcomes after the paste-submit that may leave our text in the box, the in-draft failure and the not-landed failure.
  A slash command starts no turn, so its record always types afresh; the repeat of an operator-submitted session command is stated in D5 and as new risk R8.
  The "in-draft record" is renamed the pending-send record throughout (D5, D7, D8, R6, support log), and the acceptance criterion covers the not-landed retry.
- **F3 (LOW, design) — fixed as proposed.**
  Premise confirmed: `pastePlaceholderShape` in `claudeengine/session.go` reads an optional `+<m> lines`, and `defaultSubmitConfirmTimeoutS` is 30.
  D5's bound now limits the placeholder Enter to a sent text long enough to collapse (threshold owned by claudeengine, pinned by the live check) and to a placeholder whose line count, when shown, equals the sent text's, and states that the activity check covers only the last `operator_quiet_s` before each Enter.
  R3 records the residual, an acceptance criterion pins the narrowed acceptance, and holding the check over the whole window is a rejected alternative.
- **F4 (LOW, consistency) — fixed as proposed.**
  D7's `Start` bullet splits the pre-write hold (dies after `send_ready_timeout_s`) from the foreign-box failure (dies at once, killing the pane with the draft); D7's bound, R7, Q14 and the `Start` acceptance criterion match.
- **F5 (LOW, consistency) — fixed as proposed (first option).**
  Premise confirmed: `probeIdle` in `internal/orchengine/watcher.go` clears a Stuck with `StuckByHold` on every passing probe, and the probe reads no operator activity.
  D7's bound now records a send hold under its own marker beside `StuckByHold`, left alone by a passing probe and cleared only by a passing send; the watcher acceptance criterion and Q17 match.

The `## Review rounds` section is untouched, since its entries belong to the gate.

## Disputed

None.

## Deferred

None.
