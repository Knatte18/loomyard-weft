# Card 8 — background-shell-outcome

**Edit:**
- `internal/websterengine/triage.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/triage_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/verbs_test.go`

**Intent:**
Implement decision D9 (`webster-background-shell-wait`, #397): the behavior stays — once a non-awaited background shell has been outstanding for `background_shell_wait_min`, shuttle counts Master's turn end and lyx does not stop the shell — but the `webster-background-shell` friction note and the `RunResult.Warnings` entry now state the outcome.

Per shell, the note states: the wait stopped waiting after the bound and counted Master's turn end; lyx did not stop the shell; what ends it; and the run's outcome after that turn end.
What ends it is established by the code, not guessed: on a done outcome shuttle removes Master's strand when the run finishes (`Run.finalize`, since webster's Master spec does not set `KeepPane`), which ends the session and the shell with it; on any other outcome Master's strand stays alive until the next `lyx webster run` reclaims it at entry (`reclaimEntryTimeStrands`).
The outcome is one of: done; stuck with its reason (Master's own stuck, or the verify gate's demotion); or the error the run returned (asking, died, timeout, or a mapping error).
`expiredShellWarning` produces the same per-shell wording, so the warning and the note cannot drift; it takes whatever outcome description it needs as a parameter.

In `Run`, move `writeBackgroundShellFrictionNote` from right after `handle.Wait()` to the points where the outcome is known: once the done path has its final `runResult` (after the verify-gate demotion), and on each non-done and error return, so the note is still written whatever the outcome and still best-effort (a write failure is logged, never returned).
Keep the friction note path and the `background_shell_wait_min` rendering as they are.

Tests:
`triage_test.go`: table over outcomes (done, stuck with a reason, an error) for both the note and the warning text: each names the shell label, the bound, that the turn end was counted, that lyx did not stop the shell, what ends it for that outcome, and the outcome itself.
`runlevel_test.go`: extend `TestRun_ExpiredShellYieldsWarningSummarySectionAndFrictionNote` so the note and the warning carry the done outcome and the strand-removal clause, and add a row where Master returns died with an expired shell: the note states the error outcome and that the next `lyx webster run` reclaims the strand.
`internal/webstercli/verbs_test.go`: drive `lyx webster run` through its cobra command with the fake master starter returning a died outcome and one expired shell, and assert the friction note on disk states the outcome and the reclaim clause.

**ImpactSummary:** Rewords one friction note and one warning and moves the note's write after the outcome is known in Run; no behavior of the wait or the shell changes.
