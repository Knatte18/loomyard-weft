# Round 2 fixer report — discussion

## Changes

- F1, fixed as proposed: D5's way-forward bullet now splits `status-unreadable`.
  A read-lock or raw read error that failed its re-read names re-running `lyx shed step <run> --until-stop`.
  A decode error names re-stepping with a binary that decodes the file, as after a version skew, and otherwise reporting the stop to the run's parent with the status path and loop log, as refusal-spec's `arbiter: status or envelope unreadable` row does.
  The No re-step bullet now says the file stays undecodable to the binary that failed.
  The refusal-spec bullet and the D5 acceptance row carry the same split.
- F2, fixed as proposed: D5's refusal-spec bullet states each cause's class.
  The read error that fails its re-read stays on the transient `step interrupted` row.
  `status-missing` and a decode-error `status-unreadable` go on a new shed row, `loop status missing or undecodable`, a correctness halt matching the `status file missing` row.
  Scope In, the `PATTERN-refusal-way-forward` constraint and the D5 acceptance row name the new row too.
- F3, fixed as proposed: the `verify-package-missing` acceptance row's reports-nothing list gains a Rename card whose pair's New side lies in `internal/new`, followed by a Verify on `./internal/new`.

## Disputed

None.

## Deferred

None.
