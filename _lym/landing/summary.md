# Drift: exempt parent decisions from the judge check, fix line ranges

A parent's `lym loom decision add` during a Drift-Verify judge session no longer halts the run as the judge's edit.
Drift-Expect is now told to name keyed lines by `keys` entries, so the expectations it writes match.

Closes #145
Closes #141

## What changes

### Parent decisions during a judge session (#145)

`TakeJudgeSnapshot` now also copies the decision record, named by the new `Deps.DecisionRecord`.
`JudgeSnapshot.Changes` exempts the record when all of these hold:

- the records checkout holds it committed, so an uncommitted write is still a change;
- it differs from the copy only by entries added after the Discussion;
- every entry and carry-over block the copy held is kept unchanged.

The carry-over comparison uses the new `discussionparser.CheckCarryOversNotLost`, which, unlike `CheckCarryOversKept`, does not report an added block.
A change to any other file, or any other change to the record, is named as before.
An empty `Deps.DecisionRecord` exempts nothing.

`lym loom decision add` with `--amends-plan` or a PATTERN change is refused while the run's drift scope record holds an open judge session.
Those forms write beside the record, which the check cannot tell from the judge's own edit.
The refusal says to re-run once Drift-Verify has ended the session, or to record the decision without those forms.
Plain entries are not refused.

### Line-range entries (#141)

The checker reports each changed keyed value at line 0, so no `lines` range covers it.
Rather than change the line numbers, the `lines`-mode expectation format shown to Drift-Expect now says a `lines` range counts the `line` positions of unkeyed lines and covers no keyed line.
It tells the seat to name a keyed line's value by a `keys` entry alone.

## Design decisions

- The #145 fix excludes the parent's commits from the check instead of refusing every `decision add` during a judge session.
  Only the forms the check cannot classify are refused.
- A committed entry or carry-over block that the judge itself adds to the record is also exempt.
  This is a stated bound in the `driftengine` package doc.
  An uncommitted append, an edit or removal of an existing entry or block, and a `lym loom decision replace` during the session are still named as a change.
- The #141 fix is a prompt change, not a change to the line numbers the checker reports.
  A seat that ignores it and writes a `lines` range for a keyed value still fails the gate.

## To check by hand

- No regression check ran for this task, and nothing was run against a live darn run.
  Confirm with a real Drift-Verify session that a `lym loom decision add --by parent` leaves the run going.
- Confirm that a Drift-Expect seat given the new `lines` format text writes `keys` entries for keyed values.
- Review findings left open at the last review, both fixed in the final commits but worth a look:
  - `TestDriftDeps_CarryTheDecisionRecord` in `internal/loomcli/driftwiring_test.go` needed its `testtiming` keep marker.
  - The `driftengine` package doc in `internal/driftengine/doc.go` must not describe the `loomcli` refusal, and must bound the exemption by `driftengine`'s own behavior.
