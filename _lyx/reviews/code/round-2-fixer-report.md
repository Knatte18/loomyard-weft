# Fixer-report, round 2

## Changes

- F1 (LOW): `internal/webstercli/restoreplan.go:19` now annotates `restorePlanCmd` with `clihelp.AudienceRole`, as `rebaseline` does.
  No test or generated index pins the verb under the operator audience.
  `lyx gate test ./internal/webstercli ./cmd/lyx ./internal/helpcli` passed (tier0, 491 passed, 0 failed).
  Committed as `329c5cf1`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: `restore-plan` leaves the operator index although operator-facing text names it. Disputed, ground `every fix worse` (see Disputed).

## Disputed

- fix-proposal finding (self-check), ground `every fix worse`: the index lists a command under one audience only (`internal/clihelp/index.go:54`), so keeping it in the operator index reopens F1.
  `rebaseline` already has the same dual use under `AudienceRole`, and operator text names the verb by name in `validate.go:160` and `run.go:61`.

## Deferred

None.
