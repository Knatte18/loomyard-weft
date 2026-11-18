# Drift: decision adds pass the judge check; keyed cases refuse ranges

This change fixes two drift bugs, issues #145 and #141.

## A parent decision during Drift-Verify no longer halts the judge

A parent can run `lym loom decision add` while a Drift-Verify judge session is running.
The verb commits `_lym/discussion/decision-record.md` on the records side.
When the judge finished, `JudgeSnapshot.Changes` counted that commit as the judge's edit,
and the run halted with `records: ... decision-record.md was changed`.

`TakeJudgeSnapshot` now also stores a copy of the decision record, named by the new `Deps.DecisionRecord`, beside the snapshot record.
`Changes` exempts the record when it differs from that copy only by `Added after Discussion` entries and every entry the copy held is still there unchanged.
The comparison uses `discussionparser.IdentityText` and `CheckAddedEntriesSubset`, so the check reuses the record format's own parser.
An edit anywhere else in the record, or a dropped or rewritten entry, is still reported as a change.
`loomcli`'s `driftDeps` sets the field from the run's decision record path.

Design decisions:

- The issue allowed two fixes: refuse `decision add` while a judge runs, or exclude its commits from the check.
  This change takes the second, so the parent never has to wait for a judge or come back later to record a call.
- The exemption is decided by content, not by commit, matching how the snapshot already judges paths.
  `decision add` commits carry no trailer that marks them as lym's own commits.
- Bound: an `Added after Discussion` entry or carry-over block that the judge session writes itself also goes unseen.
  A `lym loom decision replace` during the session is still reported as a change.
  The package doc states both.

## Line-range expectation entries over keyed cases are refused

When `key_patterns` keys every line of a case, each changed value is a keyed cell with line position 0.
A `lines` range counts positions among unkeyed lines only, so a range such as `"1-"` or `"7-433"` on such a case could never match.
The drift gate then failed on every changed value.

- `ParseExpectation` now refuses, under `diff: lines`, a `lines` entry for a baseline case where every line is keyed.
  The refusal says that a range counts only unkeyed lines and tells the writer to use `keys` entries instead.
  The expectation gate hands this refusal back to the expect agent, and `lym drift amend --entries` gives it to the parent.
- The expect agent's format text now says that a keyed line is named only by a `keys` entry.
  It also says that a range counts the listing's `line` positions, never file line numbers.

Design decision: the issue allowed giving cells real file line numbers instead.
This change keeps a range's meaning as positions among unkeyed lines, which unkeyed identity pairing and existing expectations depend on.
The refusal applies only when a case has keyed lines and no unkeyed line.
Refusing ranges that hold no baseline position would also refuse a valid expectation of unkeyed lines that the change adds.

## Tests

- `TestJudgeSnapshot` gains three rows.
  A committed added entry is no change.
  An edit beside an added entry is reported.
  A dropped entry is reported.
- `TestParseExpectation` gains a row for a range over a case where every line is keyed.
