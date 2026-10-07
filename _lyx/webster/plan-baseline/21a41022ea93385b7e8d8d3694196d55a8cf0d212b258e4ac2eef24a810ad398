# Card 10 — webster-asking-and-shell-note

**Edit:**
- `internal/websterengine/runlevel.go`
- `internal/websterengine/triage.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/runlevel_test.go`
- `internal/websterengine/triage_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D7 and D9 for webster: Master's asking error goes, and the `webster-background-shell` note and warning report the run's real outcome.

- `runlevel.go`: delete `MasterAskingError`, its `Error`/`Unwrap`, `ErrMasterAsking`, and the `OutcomeAsking` case of `Run`'s switch on Master's result.
  An unrecognized outcome still reaches the default case.
- `triage.go`: a turn end past an expired shell no longer produces a premature verdict, because the hold (card 5) keeps waiting until the outputs exist, the agent dies or the run times out.
  `expiredShellWarning` stops saying the wait "counted Master's turn end" as if that ended the run: it says the wait stopped waiting on the shell at a turn end, which finished the run when Master's output files existed and otherwise held it for the parent, that lyx did not stop the shell, what ends it, and the run's final outcome.
  `errorShellOutcome`'s doc and `strandReclaimed` now name died and timeout only.
- `doc.go`'s "Background shells in Master's wait" section follows the new wording and drops "asking".
- `refusal-spec.md`: the webster "Master ended early" row's trigger drops "Master asks instead of finishing", keeping a dead pane and a timeout (`PATTERN-refusal-way-forward`: the row changes in the same commit as the branch it describes).

Tests: in `runlevel_test.go`, delete the asking case and any `ErrMasterAsking` assertion;
in `triage_test.go`, rewrite the `expiredShellWarning` rows to the new wording, the error row naming died or timeout.
Leave `DeadReasonAsking` in `digest.go`/`classify.go` alone: it is recover-batch's own classification of a recovery strand, not a shuttle outcome.

**ImpactSummary:** Removes websterengine's exported asking error; the background-shell note and warnings change wording.
