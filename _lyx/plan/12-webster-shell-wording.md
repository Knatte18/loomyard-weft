# Card 12 — webster-shell-wording

**Edit:**
- `internal/websterengine/triage.go`
- `internal/websterengine/summary.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/summary_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli/verbs_test.go`

**Intent:**
Keep webster's record of `Result.ExpiredShells` true after card 11, so its friction note still records the shells (D8) without claiming every one ran past `background_shell_wait_min`.

Since card 11, `ExpiredShells` holds two kinds of shell: a transcript-reported shell the wait counted a turn end past after `background_shell_wait_min`, and a payload-reported shell still running when a gated Master's turn end found every output file present and finished at once.
Reword the three places that say every listed shell "ran past `background_shell_wait_min`" so they say what holds for both: the shell was still running when the wait counted Master's turn end, which happens after `background_shell_wait_min` for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist; lyx did not stop it.
The places are `expiredShellWarning` in `triage.go` (the run warning and friction-note line, keeping its bound and strand-ending clauses), `AppendBackgroundShells` in `summary.go` (its section heading becomes `Background shells left running` and its sentence is reworded the same way), and the `doc.go` paragraph describing them.
Nothing else in webster changes: which shells are listed, when the note is written and the outcomes it records stay as they are.

Tests: the exact-string expectations of the old wording, found by grepping the repository for `ran past` and `Background shells waited out`, move to the new wording: `summary_test.go`'s section row, `runlevel_test.go`'s `shellSentenceHead` and its summary-heading checks, and `verbs_test.go`'s comment naming the old phrase.

**ImpactSummary:** Rewords webster's background-shell warning, friction note line and summary section; their content and when they are written are unchanged.
