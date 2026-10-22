# Fixer-report, round 1

## Changes

- F1: fixed as proposed.
  `sleepjournal.Read` takes journalctl's standard error and reads an exit-0 run with empty output and a permission message there as unread, with the clause "the journal is not readable by this user".
  `machineSleeps` captures stderr into a buffer on the `exec.Cmd` and passes it through `sleepsFromJournal`.
  New `Read` rows (hint, refusal, unrelated stderr stays read) and a `TestTimingSleepNote` row cover it; `sleepjournal/doc.go`, the `Read` doc comment and `reportcli/doc.go` name the case.
- F2: fixed as proposed.
  The `sleep_note` check in `convergence_integration_test.go` now seeds a resumed hub pause in the hub's pause record and requires either a journal-read note or the note saying the recorded pauses stand in.
  No new test function.
- F3: fixed as proposed. The `sleepjournal.go` header names the read result.
- Self-check follow-up (see below): `tools/tokencount` captures journalctl's stderr and passes it to `Read`, with a test row and doc wording.

## Self-check

- fix-consistency, MEDIUM: `tools/tokencount` passed nil stderr to `Read`, leaving the permission case reported as no sleep. Fixed as proposed (own change beyond the review's named files).
- fix-consistency, LOW: no permission row in `tokencount/sleep_test.go`. Fixed with the row above.
- fix-affected-set, MEDIUM: same `tokencount` finding, merged. Fixed.
- fix-code-correctness, LOW: the integration check only constrains the note on hosts where the journal is unread, since the parallel test cannot change PATH. Accepted as the review's own scoped fix; closing it needs an injectable journalctl runner, a larger change the review did not ask for.
- fix-proposal, LOW: same integration residual, merged. Accepted as above. Its speculative note on systemd rewording the hint is not a finding.

## Disputed

None.

## Deferred

None.
