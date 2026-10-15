# Fixer report — discussion round 1

## Changes

All in `_lyx/discussion/decision-record.md`.

- **F1** (fixed as proposed, second branch): D4 now states that with no snapshot a batten tackler's writes in the prime (files, index, a local commit) go unchecked and a resume may re-enter Land or Worktree-Teardown over them, and that the only bound is the stencil's write rule plus shuttle's autonomous-posture denies; the stop-key check is named as bounding run movement only. D4's Bound line carries the write bound. New acceptance criterion: the rendered batten tackler prompt allows writes to the verdict and case-brief paths only. The narrowed-snapshot branch was not taken, since D4's own reason (other writers in the prime) makes any prime snapshot read `tampered` falsely; `tacklerSpec` in `internal/loomcli/tacklerprompt.go` sets no write restriction beyond the autonomous posture, so the stated bound is the real one.
- **F2** (fixed as proposed, option a): D5's Migrate sentence is replaced. It names that `Migrate` runs on every reconcile and on `run_settings` (`internal/configreg/runsettings.go:60`), and makes the migration one-time and hub-level: it fires only when all four keys equal the old defaults together with the old template's `driver_kind` comment, and it replaces that comment, so it never refires and never touches a `run_settings` document. D5 now states that the loom template's four, `darn.yaml`'s four and `batten.yaml`'s three keys ship with the inherit value. The Migrate acceptance criterion covers the second reconcile, an off-default file and the `run_settings` case; a fresh-hub criterion was added.
- **F3** (fixed as proposed): D5 says the arming module passes the run's own recipe's override for every tackler key, picked by the seed's recipe in `lyx loom drive`'s wiring. Acceptance criterion added for the darn and loom `tackler_max_spawns` swap.
- **F4** (fixed as proposed): D2 gains the resume-goto-targets seam; D4's resume sentence routes through it; D7 says the stencil's goto rule names that seam's rows. The batten acceptance criterion now refuses a routed-from goto, and a prompt criterion checks the halted row is the only goto target.
- **F5** (fixed as proposed): D2's Bound line rewritten to the fix key's text.
- **F6** (fixed as proposed): D3's premise sentence replaced; D4 gains a sentence for a batten run started on the old build.
- **F7** (fixed as proposed): acceptance criteria added for batten's never-firing hand-over and verify-goto rows, and for the rendered prompt naming no arbiter and carrying the stop-kind list and reading order.
- **F8** (fixed as proposed): chose inherit. A module's `tackler_timeout_min: 0` inherits the next layer; a `shed.yaml` 0 takes 10 minutes. The plan's representation note now covers `tackler_max_spawns` only, since a timeout 0 no longer needs telling from unset. Added to the overrides acceptance criterion.
- **F9** (fixed as proposed): the duplicate `abandonedSession` clause is cut and the list names `lyx batten run`'s envelope.

## Disputed

None.

## Deferred

None.
