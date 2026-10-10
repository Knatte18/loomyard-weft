# webster: survive replans, sandbox forks, record every ended shell

Webster stays consistent when a plan is replanned after it recorded state, its batch and fabric checks judge only what they should, implementer forks cannot drive the real hub's fabric, and the shell wait leaves nothing unrecorded.

## What changes

- **Replan after webster state (#466).**
  The Webster row rebaselines a changed plan on entry instead of hard-failing on a plan fingerprint mismatch.
  A follow-up card added after the last begun batch continues over the kept batches, with a warning naming the rebaseline.
  An edit to a done card, a begun batch's card-set change, an uncheckable failed batch or an overview change outside the Card Index blocks the row as Stuck with the rebaseline's refusal.
  `lyx webster run` on that worktree still refuses with the fingerprint text.
- **Plan-Review over a begun run (#465).**
  The rubric tells reviewer and fixer which batches are begun and which cards are done, and says follow-ups land through webster.
  The plan gate refuses an edit to a done card (`done-card-edited`).
- **Rebaseline over an empty overview frame** logs the fallback and its refusal names the cause.
- **`commitsNamedBy`** classifies every candidate in one `git cat-file --batch-check` call, through the new checked `gitexec.RunStdin`.
- **`verify-nested-module` and `verify-module-wide`** share one chain splitter that splits at `||` and `|` as well.
- **`record-batch` done-checks (#484)** see the batch's own cards only; refusals open with the batch id and its card list.
- **Fabric sandbox (#494).**
  `lyx fabric add`, `clone`, `push` and `sync`, and the other mutating verbs, refuse from a webster, recovery or conflict strand name.
  The implementer stencil states the sandbox rule, and `lyx fabric add --help` says the verb pushes to origin.
  `lyx gate test` and `verifytree.Verify` drop `LYX_STRAND_NAME` from their tests' environment.
- **Read-only fabric references (#493).**
  The reference audit treats a read-only command as a warning, through an allow-list classifier of plain-word forms (`IsReadOnlyCommand`).
  `recover-batch` accepts a failed batch whose uncheckable findings are all read-only, reporting `audit_not_acceptable` on the envelope.
- **Shell wait (#496).**
  Transcript-reported and payload-reported background shells follow one rule, keyed on the output files.
  Every shell a run ends over is recorded in `Result.EndedShells`, logged, and listed in webster's summary with signal and duration.
- **Round gate re-run (#497)** names the top-level test only, so subtests of one test emit one `-run '^TestA$'` step.
- **Dead-recovery retry (#498).**
  Recovery spawns are counted, a dead recovery that committed gets one more, and the cap is two.
  `recover-batch` and `status` carry the count and `recovery_retry`; `recover-batch` refuses with `recovery_exhausted`; `reset --to batch-start` clears the count.
  The Master stencil has split dead rungs and an exhausted rung.
- **PATTERN entry lines (#476)**: a card whose fenced block holds a `PATTERN-` entry line over the cap fails `pattern-entry-line-cap` at the plan gate.
- **`TestLoomResumeReconcilesTheHubConfigBeforeArming` (#492)** did not reproduce a failure, alone or under CPU load, so only its doc comment changed.

Closes #466
Closes #465
Closes #476
Closes #484
Closes #494
Closes #493
Closes #496
Closes #497
Closes #492
Closes #498

## Design decisions

- The auto-rebaseline refusals never tell the operator to run `lyx webster rebaseline`; they name re-stepping the row.
- The fabric guard checks the strand name and the exported variable, so a fork that clears the variable, types the hidden bypass flags on `push`, or hides a command behind `bash -c` is caught by review, not lyx.
- The read-only classifier is an allow-list of plain words, so a quoted or globbed read such as `git log --format='%h %s'` stays a correctness halt.
- The guard also refuses the conflict resolver's role; a landing whose resolver needed a fabric verb would meet the refusal.
- A stale shell of either signal now holds a run until its own timeout, where a transcript-reported one expired after ten minutes before.
- After a crash, an edit to a card of the in-flight batch is adopted as an amendment on row entry, without an operator naming it.

## To check by hand

- `lyx webster` after a goto back to Plan-Bouncer with a follow-up card, and with an edited done card, behaves as described above; this was not run end to end.
- Integration suites for the touched packages were run by the verify step only for `webstercli`, `loomcli`, `battencli`, `lyxcwd`, `testkit/envkit` and `testkit/shedfake`; run `lyx gate test --tags integration ./...` before pushing.
- The fixer reports `battencli`, `lyxcwd`, `testkit/envkit` and `testkit/shedfake` pass under `--tags integration`; confirm `refscanner.go` belongs in `enforcementAllowlist`.
- Open findings to check:
  - the classifier rows for fabric references in the plan miss glob and tilde forms (`card14-classifier-rows-miss-glob-and-tilde`);
  - a failure of the Merriam base escapes Stuck on the card 4 path (`card4-merriambase-failure-escapes-stuck`);
  - the tests for the cards on the done-check, classifier, resume and PATTERN-cap paths are thin (`thin-test-rows-cards-13-11-17-21`);
  - a stale comment about waited-out shells in the loop-agreement code (`loop-agrees-stale-waited-out-comment`).
- Master's turn end was counted while three background shells were still running in the session; confirm no stray test process remains.
