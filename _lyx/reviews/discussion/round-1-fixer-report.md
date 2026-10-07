# Round 1 fixer report

All eight findings fixed in `decision-record.md` and `support-log.md`; `lyx loom validate-discussion` exits 0 on the result.
No tests were added: the target is a design artifact, and every finding was a defect in a Decision, not a coverage gap.

## Fixed

- **F1 (D3, Windows):**
  D3 now kills the process group on Unix and the shell only on Windows, logged as degraded; `WaitDelay` bounds the pipe read on both.
  The acceptance criterion is split by platform.
  The open "must hold or be stated" Constraint bullet is removed, including its pane-options clause, since reed already drives `set-option -p`.
  The Job Object alternative is in the support log's rejected alternatives, with Q12.
- **F2 (D5, gated strands):**
  Shuttle sets the done-when list on an ungated run's strand only.
  D5's Bound now says a gated strand is relaunched as before, so no gate loses its re-prompt, and its Rationale says outputs mean finished for an ungated run.
  The Open risk is deleted, and an acceptance row covers a gated strand with all outputs present.
  The Constraint line for `PATTERN-completion-signal` follows.
  Rejected alternative and Q13 recorded.
- **F3 (D7):**
  The outcomes are ordered by run state.
  They now cover no status file, done, awaiting, a running run (no-op on a live strand or a held run lock, refused otherwise, so a Go-driven run is handled without reading the seed), the unfinished-merge refusal and the shared bootstrap lock.
  The acceptance criterion lists the same set.
- **F4 (D8, PATTERN):**
  D8 now states `PATTERN.md` changes in the same commit: `PATTERN-completion-signal` and its background gain the done-when list, and one new entry covers the display-only wait mark.
  The "no PATTERN entry is added" clause is removed.
- **F5 (D1, D2, stale mark):**
  D2's Bound now says what a stale or misdirected mark can show and that it never hides a real wait.
  The landing callback clears the driver strand's mark at the start of every verify, and a callback whose strand is gone is a logged no-op.
  D1 states first live marker wins.
  The Open risk names the driver strand, and an acceptance row covers the callback.
- **F6 (D6):**
  D6's state list reads "a halted or awaiting child whose driver strand is not live".
  Its Bound covers the re-spawn.
  The way forward after a failed revive agrees with D7's.
  The Constraint no longer calls batten's notice a refusal with a spec row.
  The notice wording stays inside the one clause D6 adds; I judged that this does not enter the Scope Out item for notice wording.
- **F7 (Goal, acceptance):**
  The Goal's "every Go-side wait" and "only strands with work left" are reworded to match Scope and D5.
  The status-line criterion states the start-time fallback.
  The D6 acceptance row covers awaiting children.
- **F8 (relocation):**
  The `bugfix-webster` narrative (Scope, D1 Rationale, first Auto-mode assumption), the observed verify duration, the dated incident and the dated operator attributions are cut from `decision-record.md`.
  The duration moved into Q3, the `bugfix-webster` detail into Q1, and the attributions into new Q11; the completeness check found each requirement already stated in a Decision.
  The D8 package list is reduced to "each touched package's `doc.go`".

## Deferred

Nothing.
