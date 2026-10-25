# Round 3 fixer report — discussion

Each finding's premise held against the code: `battenPreRun` calls `refuseCovered` first (`internal/battencli/arm.go:644`), `parentWarning` judges every entry (`internal/adoptcli/adopt.go:385`), `set-status` refuses an absent `status` key (`internal/boardcli/cli.go:642`), and `UpsertTask` has no kind-change guard.

## Changes

- F1, BLOCKING: fixed differently.
  The review proposed that dobby run no pass at all of a covered run.
  The self-check's fix-proposal lens showed that this drops `check()`'s `lockStep` and `noticeStep` under a pause (`internal/battenshed/innerrun.go:700-723`), so an idle child's parent lock is never released and a sibling's Finalize fails.
  So D4 now has `--continue-pass` skip `refuseCovered`, and continue passes run under a covering pause.
  Dobby runs no full pass of a covered run, bar the one at the lift.
  A full pass refused at pre-run for a pause exits with a code of its own, which dobby reads as covered: no failing pass, no D7 marker and no refused-start path.
  A hand pass keeps the `lyx resume` refusal.
  A run owed a full pass under a pause is an accepted bound.
  With continue passes under a pause, the spawn hold and the owner re-check are reachable as D4 had them.
  The acceptance bullets on coverage, the spawn hold and the adopted-back respawn were rewritten to match, and a covered-run bullet was added.
  The same change also touched D4's `--continue-pass` bound, D5, D7 and D13.
- F2, MEDIUM: fixed as proposed.
  A full pass that resumes with origin unreachable writes a per-run origin-owed marker under the run's scratch directory.
  A full pass that reaches origin and exits 0 removes it.
  Dobby reads the marker instead of an output line.
  The `--continue-pass` bound, D3 and the acceptance bullets were updated to match.
- F3, MEDIUM: fixed as proposed.
  A failed owner pause is no longer retried blind: dobby runs a full pass on each probe tick while the mark stands and no pause covers the slug.
  The acceptance bullet now says that a run adopted back while its owner pause had failed is not paused.
  D4's in-flight line and D13 now count such a run for the backoff.
- F4, MEDIUM: fixed as proposed, then extended by the self-check.
  The D2 refusal now covers a task or a beam.
  Both refusals are decided before the owner write, against the entries that passed step 1, since `parentWarning` runs after `SetOwners` today.
  Each refusal names one call that meets both rules.
  A parent lost to a race inside the batched write is an accepted bound.
  The Scope, the acceptance bullet and Q21 were updated.
- F5, MEDIUM: fixed as proposed.
  The release is now `lyx board set-status '{"slug":"<slug>","status":null}'` in D5, in the start-guard refusal and in the acceptance bullet; Q6 was corrected.
- F6, MEDIUM: fixed differently.
  The proposal had "a write that leaves an entry of kind task with an empty status gives it `draft`, unless the write is a release" as its rule.
  That rule would re-draft a released task on any later edit that leaves its status untouched.
  The rule therefore also spares a write to an entry that was already a task and leaves its status untouched.
  Empty means nil or an empty string, everywhere, and an empty-string status is never a release.
  After the fix-proposal lens, D5 also states that no lyx code path writes a null task status.
  The acceptance bullet names the project-to-task upsert, the empty-string status and the released-task edit, and the closing claim is restated as a consequence of the rule.

The support log records round 3 under Review rounds.

## Self-check

- fix-claims, MEDIUM, `parentWarning` runs after the owner write, so it cannot be the refusal as written: fixed, both D2 refusals are decided before the owner write against the entries that passed step 1 (merged with fix-way-forward's finding).
- fix-claims, LOW, `lyx board promote` cannot turn a project into a task: fixed, D5 now says a note promoted by `lyx board promote`.
- fix-consistency, MEDIUM, a start refused for a racing pause still takes D5's refused-start notice and backoff: fixed, D5 exempts it.
- fix-consistency, LOW, the `--continue-pass` acceptance bullet lacks the marker case: fixed.
- fix-consistency, LOW, the restart criterion and D13's in-flight definition ignore covered runs: fixed, D13 defines in flight as a run dobby may pass on its next probe tick, and the restart criterion names the lift.
- fix-consistency, LOW, a marked run with a failed owner pause backs off under D13: fixed, such a run is in flight for the backoff while no pause covers it.
- fix-consistency, LOW, D3 says dobby stops passing a taken-over run, and its list of full passes lacks the marker: fixed.
- fix-way-forward, MEDIUM, a parent lost inside the batched owner write slips past the refusal: fixed, the refusal is decided before the write, and the race is an accepted bound under today's warning.
- fix-proposal, MEDIUM, passing no covered run holds an idle parent lock and the notices until the lift: fixed differently from both options the lens offered: continue passes skip `refuseCovered`, and only an owed full pass under a pause waits, as an accepted bound.
- fix-proposal, MEDIUM, chained adopt refusals each name a call that is refused again: fixed, each refusal names one call with every slug both rules reach.
- fix-proposal, LOW, an internal null status write would count as a release: fixed, D5 states that no lyx code path writes a null task status.

## Disputed

None.

## Deferred

None.
