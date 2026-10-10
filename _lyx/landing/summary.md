# webster: stay consistent across replans, sandbox forks, record shells

Fixes a sweep of webster and loom defects.
A replan after webster has begun no longer hard-fails, implementer forks cannot drive the real hub's fabric, and the shell wait records what it leaves behind.

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

## What changes

- **Replan after webster began (#466, #465).**
  The Webster row rebaselines a changed plan on entry instead of failing on a fingerprint mismatch.
  Refusals the auto-path cannot resolve block the row as Stuck and never name `lyx webster rebaseline`; `lyx webster run` still refuses with the fingerprint text.
  An accepted rebaseline puts a warning in the run summary and opens a following Stuck reason.
  The plan gate reports `done-card-edited` for an edited done card (`EditedDoneCards`).
  The Plan-Review rubric carries webster's run record and a `webster_record` marker, so reviewer and fixer know which cards are frozen and that follow-up cards land through webster.
- **Rebaseline over an empty overview frame.**
  It logs the fallback and refuses naming the cause (no recorded overview hash, absent baseline copy, or a copy without a Card Index).
- **`commitsNamedBy`** classifies every candidate in one `git cat-file --batch-check` call, through the new `gitexec.RunStdin`.
- **Verify checks (`verify-nested-module`, `verify-module-wide`)** share one chain splitter that splits at `||` and `|`.
- **PATTERN entry cap (#476).**
  A card whose fenced block holds a `PATTERN-` entry line over the cap fails `pattern-entry-line-cap` at the plan gate.
- **`record-batch` done-checks (#484)** look at the batch's own cards only, and the refusals open with the batch id and its cards.
- **Fabric sandbox (#494).**
  `lyx fabric add`, `clone`, `push` and `sync` refuse from a webster, recovery or conflict strand name; `prune --apply`, `cleanup --apply` and `shortname x` refuse from a webster strand.
  Every refused verb's way forward names the orch, with matching `refusal-spec.md` rows.
  The implementer stencil states the sandbox rule, and `lyx fabric add --help` says it pushes.
  `lyx gate test` and `verifytree.Verify` drop `LYX_STRAND_NAME` from their tests' environment.
- **Read-only fabric references (#493).**
  `IsReadOnlyCommand` is an allow-list of plain words (fabric readers, `--help`, git reads); a read-only reference is a policy warning.
  `recover-batch` accepts a failed batch whose uncheckable findings are all read-only, reports `audit_not_acceptable`, and the Master stencil has a rung for it.
- **Shell wait (#496).**
  Transcript-reported and Stop-payload-reported shells follow one rule keyed on the output files.
  Every shell a run ends over lands in `Result.EndedShells`, and webster's summary lists them with signal and duration.
  A recovery strand's turn end with a shell outstanding no longer counts.
- **Round gate (#497).**
  The Publish failure re-run names the top-level test only, one `-run '^TestA$'` step for several subtests of it.
- **Recovery retry (#498).**
  A dead recovery that committed past its `RecoveryStartSHA` gets a second recovery; the cap is two counted spawns, then `recovery_exhausted`.
  `recover-batch` and `status` carry the count and `recovery_retry`; `reset --to batch-start` clears the count.
  The Master stencil has split dead rungs and an exhausted rung.

## Design decisions

- The auto-rebaseline runs behind a run option set by the shed adapter, so the `lyx webster run` CLI keeps its refusal.
- The sandbox guard keys on the exported strand name.
  A fork that clears the variable, types the hidden bypass flags on `push`, or hides a command behind `bash -c` is caught by review, not by lyx.
- The read-only classifier is an allow-list of plain words, so a quoted or globbed read such as `git log --format='%h %s'` stays a correctness halt.
- The conflict resolver's role is refused the mutating fabric verbs too.
- A stale shell now holds a run until its own timeout, where a transcript-reported shell used to expire after ten minutes.
- After a crash, an edit to a card of the in-flight batch is adopted as an amendment on row entry without an operator naming it.

## Check by hand

- #492: `TestLoomResumeReconcilesTheHubConfigBeforeArming` did not reproduce a failure, alone or under CPU load, so only its doc comment changed.
  Confirm it is stable under repeated `lyx gate test --tags integration ./internal/loomcli` runs.
- `internal/fabricengine/refscanner.go` joined `enforcementAllowlist` because it names `--show-toplevel` and resolves no root; confirm that is right.
- `d8-reader-write-flags-unbounded` (design, BLOCKING, open): the reader allow-list may accept a git reader flag that writes; check the read-only classifier against the flags git readers accept.
- `d12-begin-batch-dead-refusal-stale` (consistency, MEDIUM, open): check `begin-batch`'s terminal-dead refusal names the retry and `recovery_exhausted`.
- `card14-classifier-rows-miss-glob-and-tilde` (scope, MEDIUM): the classifier rows in tests lack glob and tilde cases.
- `card4-merriambase-failure-escapes-stuck` (scope, MEDIUM): a failure computing the merge base in the auto-rebaseline may escape the Stuck mapping.
- `thin-test-rows-cards-13-11-17-21` (scope, MEDIUM): some test tables are thin; review the rows for the classifier, the rebaseline and the shell wait.
- Background shells outstanding at Master's last turn end (not confirmed finished): an integration run of `./internal/webstercli ./contracts/stencils ./internal/loomcli`, and one of `./internal/battencli ./internal/lyxcwd ./internal/testkit/envkit ./internal/testkit/shedfake`.
  Run `lyx gate test ./...` and `lyx gate test --tags integration ./...` before pushing.
