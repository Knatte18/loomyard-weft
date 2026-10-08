# Card 12 — webster-shell-wording

**Edit:**
- `internal/websterengine/triage.go`
- `internal/websterengine/summary.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/summary_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/triage_test.go`

**Intent:**
Keep webster's record of `Result.ExpiredShells` true after card 11, so its friction note still records the shells (D8) without claiming every one ran past `background_shell_wait_min`.
This is D8's own consequence, not added scope: D8 returns a done-at-once shell in `ExpiredShells` "so webster's friction note still records them", and the texts below would otherwise state something false about that shell; the edit is limited to the sentences that become false.

Since card 11, `ExpiredShells` holds two kinds of shell: a transcript-reported shell the wait counted a turn end past after `background_shell_wait_min`, and a payload-reported shell still running when a gated Master's turn end found every output file present and finished at once.
Reword the sentences that say every listed shell "ran past `background_shell_wait_min`" so they say what holds for both: the shell was still running when the wait counted Master's turn end, which happens after `background_shell_wait_min` for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist; lyx did not stop it.
The sentences are `expiredShellWarning`'s in `triage.go` (the run warning and friction-note line, keeping its bound and strand-ending clauses), the section sentence `AppendBackgroundShells` writes in `summary.go` (its `Background shells waited out` heading stays as it is, since the wait stopped waiting on both kinds of shell), and the `doc.go` sentence saying every other background shell is waited out after `background_shell_wait_min`.
In `expiredShellWarning` only the `ran past <bound>` head is replaced; these stay byte-identical: the bound with its minutes (`` `background_shell_wait_min` (15 minutes) ``), the clause `the wait stopped waiting on the shell at a turn end, which finished the run when Master's output files existed and otherwise held it for the parent`, `lyx did not stop the shell`, and the strand-ending and final-outcome clauses, so `webstercli/verbs_test.go`'s `TestRunCmd_DiedMasterNotesExpiredShellOutcome` stays valid unchanged.
Nothing else in webster changes: which shells are listed, the section heading, when the note is written and the outcomes it records stay as they are.

Tests: the exact-string expectations of the reworded sentences move to the new wording: `summary_test.go`'s section row and `runlevel_test.go`'s `shellSentenceHead`; the summary-heading checks keep `Background shells waited out`.
`triage_test.go`'s `TestBackgroundShellNoteAndWarning` keeps its required substrings, which the kept clauses satisfy, and its list gains the new head's wording in place of `ran past`, with the test's doc comment naming it.

**ImpactSummary:** Rewords the sentences of webster's background-shell warning, friction-note line and summary section that card 11 makes false; headings, content and when they are written are unchanged, and only websterengine's own tests assert the wording.
