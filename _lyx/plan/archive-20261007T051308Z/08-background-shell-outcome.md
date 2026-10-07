# Card 8 — background-shell-outcome

**Edit:**
- `internal/websterengine/triage.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/triage_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/verbs_test.go`

**Intent:**
Implement decision D9 (`webster-background-shell-wait`, #397): the behavior stays — once a non-awaited background shell has been outstanding for `background_shell_wait_min`, shuttle counts Master's turn end and lyx does not stop the shell — but the `webster-background-shell` friction note and the `RunResult.Warnings` entry now state the outcome.

Per shell, the note states: the wait stopped waiting after the bound and counted Master's turn end; lyx did not stop the shell; what ends it; and the run's outcome after that turn end.
What ends it is established by the code, not guessed, and it follows the shuttle outcome: when Master's turn ends shuttle-done, which covers a webster done, Master's own stuck or paused and the verify gate's demotion alike, shuttle removes Master's strand when the run finishes (`Run.finalize`, since webster's Master spec does not set `KeepPane`), which ends the session and the shell with it; when the run returns an error (asking, died, timeout) Master's strand stays alive until the next `lyx webster run` reclaims it at entry (`reclaimEntryTimeStrands`), and a mapping error after a shuttle-done end takes the removal clause.
The outcome is one of: done; stuck with its reason (Master's own stuck, or the verify gate's demotion); paused (`mapMasterDone` returns it as `RunResult.Outcome`); or the error the run returned (asking, died, timeout, or a mapping error).
`expiredShellWarning` produces the same per-shell wording, so the warning and the note cannot drift; it takes whatever outcome description it needs as a parameter.

In `Run`, move `writeBackgroundShellFrictionNote` from right after `handle.Wait()` to the points where the outcome is known: once the done path has its final `runResult` (after the verify-gate demotion), and on each non-done and error return, so the note is still written whatever the outcome and still best-effort (a write failure is logged, never returned).
The `RunResult.Warnings` entries move with it: today `Run` appends them before the verify-gate demotion and only when `runResult.Outcome` is done, so a demoted run would warn "done" while the note says "stuck", and a Master-stuck run would get no warning.
Build them after the demotion, from the same outcome description the note uses, and append them on every outcome that returns a `RunResult` (done, stuck and paused); an error outcome returns no `RunResult`, so the note alone carries it.
`AppendBackgroundShells` and the summary section keep their done-only condition.
Keep the friction note path and the `background_shell_wait_min` rendering as they are.

Docs: rewrite `doc.go`'s "Background shells in Master's wait" paragraph, which says the warning appears "On a done outcome", quotes the old warning text and says a non-done outcome keeps only its own error or stuck reason; it now says the note and the warning state the outcome, the warning is on every outcome that returns a `RunResult` and the summary section stays done-only.

Tests:
`triage_test.go`: table over outcomes (done, stuck with a reason, paused, an error) for both the note and the warning text: each names the shell label, the bound, that the turn end was counted, that lyx did not stop the shell, what ends it for that outcome, and the outcome itself.
`runlevel_test.go`: extend `TestRun_ExpiredShellYieldsWarningSummarySectionAndFrictionNote` so the note and the warning carry the done outcome and the strand-removal clause, and add rows where Master writes stuck with an expired shell (note and warning both state stuck with its reason and the strand-removal clause) and where Master returns died with an expired shell (the note states the error outcome and that the next `lyx webster run` reclaims the strand).
`internal/webstercli/verbs_test.go`: drive `lyx webster run` through its cobra command with the fake master starter returning a died outcome and one expired shell, and assert the friction note on disk states the outcome and the reclaim clause.

**ImpactSummary:** Rewords one friction note and one warning and moves the note's write after the outcome is known in Run; no behavior of the wait or the shell changes.
