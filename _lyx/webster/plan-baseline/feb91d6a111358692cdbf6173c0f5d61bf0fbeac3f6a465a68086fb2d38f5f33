# Card 2 — shell-result-unread-wording

**Edit:**
- `internal/websterengine#expiredShellWarning`
- `internal/websterengine#AppendBackgroundShells`
- `internal/websterengine#masterAwaitedShellPrefix`
- `internal/websterengine#Run`
- `internal/websterengine/doc.go`
- `internal/websterengine/triage_test.go`
- `internal/websterengine/summary_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/verbs_test.go`

**Intent:** Reword webster's record of the shells a run ended over to say the shell's result was never read by the run (decision D1).
`expiredShellWarning`, which is both the `RunResult.Warnings` entry and the `webster-background-shell` friction note's line, keeps the shell's label, the signal that reported it and its time outstanding ("outstanding for <d>"), and says the shell was still running when the run ended, that the run never read its result, that lyx did not stop the shell, what ends it and the run's final outcome.
`AppendBackgroundShells`' section intro says Master's run ended while these shells were still outstanding and their results were never read by the run.
Reword the doc of `masterAwaitedShellPrefix` and the comment beside `AwaitedShellPrefixes` in `Run`'s Master spec: the prefix keeps the backgrounded recovery verb out of the background-shell wait marker; it no longer changes when a turn end counts, since every shell holds a gated turn end alike.
Update the "Background shells in Master's wait" section of `doc.go`: a gated turn end waiting on a shell is never an arrival; the run ends on the next turn end with nothing outstanding, the run deadline, the liveness check or a stop; a deadline still ends it done through shuttle's satisfied file contract with the gate run once at finalize, and the shell's result is then never read; the warning, the note and the summary section say so.

Tests: update the wording assertions in `TestBackgroundShellNoteAndWarning` (`triage_test.go`), the expected section text in `summary_test.go`, and the `shellSentenceHead` expectations of `TestRun_ExpiredShells` (`runlevel_test.go`), each asserting the new "never read" phrase.
In `TestRunCmd_DiedMasterNotesExpiredShellOutcome` (`internal/webstercli/verbs_test.go`), which reads the `webster-background-shell` friction note, add the "never read" phrase to the asserted substrings; its "outstanding for 1m30s" and label assertions stay true.
Grep the repository for the old phrases "may still be running in the session", "was still running, outstanding for" and "outstanding for" and update every test that asserts them.

**ImpactSummary:** Wording only in websterengine's warning, note and summary section, read by tests in websterengine and webstercli; no decision changes.
