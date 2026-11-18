# Drift-Verify: no false guard halt on record edits, shorter halt note

Two Drift-Verify halts from task `dop-pumps-off-negative-spp` are fixed in `internal/driftengine`, with the wiring in `internal/loomcli`.

Closes #146
Closes #147

## Judge guard and decision-record edits (#146)

The judge-session guard halted a run when the parent edited the decision record with `lym loom decision add` or `lym loom decision replace` while the judge ran.
A resume then repeated the first halt's text, so the operator could not tell whether a restore had taken effect.

- The judge snapshot and the guard comparison now hold the run's decision lock, so they never read a decision verb's write before its commit.
  `Deps.LockDecisionRecord` is the seam, wired in `internal/loomcli/driftwiring.go` to the lock the `lym loom decision` verbs hold.
- A decision record is no judge change when it differs from its base only by entries added after the Discussion.
  The base is the snapshot's copy, or the record as the newest `lym loom decision replace` commit wrote it, provided that commit came after the snapshot's records head.
  A replace that predates the snapshot is no base.
  `Deps.ReplacedDecisionRecord` reads it, and `fabricengine.RecordsCommitIsAfter` orders the commits.
- Each change in the halt note now carries the file's modification time against the judge session's window.
  A file modified after the guard first halted the returned session reads as changed by another process.
  A change with no file, such as a moved HEAD or a deleted path, carries no time.
- The guard check is recorded on the session mark as `GuardHalt`.
  A resume that re-checks says what still differs and what was restored since the last check, instead of repeating the first halt.
  A check that finds nothing clears the record, and a check over records that differ from their commit writes nothing.

## Halt note for judge findings (#147)

The halt reason held one sentence per changed line, each with the same rationale, and put the way forward last.

- Rulings that share a case, key, kind and reason become one finding.
  It names the row range, the row count, and the old and new values of the first and last rows.
  Rows are a key's occurrences for a keyed group and line positions otherwise.
  A group of one, and an added or removed case, keep their own wording.
- The halt reason opens with the lead, the finding count and the way forward.
  It then carries the findings, capped at 1500 characters, whole findings only, with an `and N more` note for those left out.
  A first finding that alone passes the cap is cut with an ellipsis.
- Every finding, ungrouped, is written to `verify-findings.md` under the drift logs directory (`Paths.Findings`), which the reason names by its worktree-relative path.
  The logs directory is outside the judge guard, so the write does not trip it.
- Stuck halts over a failing compare and over stored case rulings use the same format.

## Design decisions

- The decision record stays in the guarded set; a lock plus a base comparison replaces an exemption.
  An exempt record would let a judge edit pass, and the comparison still rejects any change that is not an entry added after the Discussion.
- The modification-time wording is evidence for the reader, not a verdict: an edit inside the window may be the judge's or a process beside it.
- The cap counts separators and the omission note, so the reason never passes the limit.

## To check by hand

- No run against a real `lym loom` Drift-Verify halt was made; the change is covered by unit tests and the record-history integration test in `internal/loomcli`.
- Confirm that a parent's `lym loom decision add` made while a judge runs no longer halts the run, and that a judge's own edit of the record still does.
