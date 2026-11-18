---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/driftengine/expectation.go:297
    summary: The new refusal rejects a valid expectation, a range over an unkeyed line the change adds to a baseline case whose every line is keyed, and no keys entry can cover that cell instead.
    fix: >-
      Remove the refusal from checkCase in internal/driftengine/expectation.go (the unkeyed counter and the `unkeyed == 0 && len(occurrences) > 0` branch),
      and drop the two sentences it added to the ParseExpectation doc comment and the two it added to internal/driftengine/doc.go under expected.yaml.
      Keep the new sentence in the linesExpectationFormat text in internal/driftengine/seats.go, which builds the issue's second option by telling the expect agent to name keyed values by keys entries.
      In TestParseExpectation in internal/driftengine/expectation_test.go, replace the RangeOverKeyedCase row with an accepted row giving a `lines` range on the fully keyed baseline case keyed.txt,
      and add a fully keyed case plus a mixed case to the test's baseline only if a row needs them.
      Rewrite the line-range section of _lym/landing/summary.md so it describes the format-text change alone.
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/driftengine/check.go:244
    summary: An expectation sealed before this change with a range over a fully keyed case now fails every drift check with the sealed expectation does not parse, and the amend workaround from issue 141 can no longer clear it.
    fix: >-
      Removing the refusal under F1 settles this, and no further change is needed then.
      If the fixer keeps any form of the refusal instead, apply it only where expected.yaml is first parsed, in internal/driftengine/expect.go at the ParseExpectation call, and in internal/driftengine/amend.go,
      and never when sealedAndAmended in internal/driftengine/check.go re-parses the sealed bytes;
      add a TestParseExpectation or check-level row showing a sealed expectation with such a range still parses.
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/judgesnapshot.go:331
    summary: The decision-record exemption passes more than the parent's verb writes, since an uncommitted entry the judge appends and any edit or removal of an existing carry-over block are both exempt.
    fix: >-
      In appendedDecisionRecord in internal/driftengine/judgesnapshot.go, also require every carry-over block of the snapshot's copy to be present and unchanged in the current record, adding to internal/discussionparser an exported check that reports only dropped or changed blocks if CheckCarryOversKept's added-block findings get in the way.
      Also grant the exemption only when the records checkout's current state does not list the record among its changed or untracked paths, so a judge's uncommitted append is named; pass that state in from Changes, which already holds it.
      Rewrite the bound in internal/driftengine/doc.go and in _lym/landing/summary.md to state what still goes unseen after the change.
      Add TestJudgeSnapshot rows for an uncommitted appended entry, which is named, and for an edited carry-over block, which is named.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/driftengine/judgesnapshot.go:270
    summary: Two forms of lym loom decision add still halt the judge, the --amends-plan form that commits _lym/loom/amendment.yaml in a tapestry run and the PATTERN-entry form that inserts or withdraws a bullet in the record outside the added entries.
    fix: >-
      Make the issue's expected outcome hold for every form of the verb.
      Either extend the exemption in Changes and appendedDecisionRecord so the amendment record's appended titles and the record's PATTERN-section bullet the verb writes beside its entry pass,
      or refuse these forms in internal/loomcli/decision.go while the drift scope record's judge session is open, with a refusal whose way forward says to re-run the call after Drift-Verify ends and a test that reaches it.
      State the result in internal/driftengine/doc.go and add a TestJudgeSnapshot row, or the loomcli refusal test, for each form.
    origin: lens:code-scope
  - id: F5
    severity: LOW
    class: design
    location: internal/loomcli/driftwiring.go:249
    summary: No test pins that driftDeps carries the run's decision record path into Deps.DecisionRecord, so dropping the line turns the issue 145 fix off with every test passing.
    fix: Add an assertion to an existing internal/loomcli test that builds driftDeps with a driftTables whose decisionRecordPath is set, checking that the returned Deps.DecisionRecord equals it.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/driftengine/judgesnapshot_test.go:323
    summary: TestJudgeSnapshot has no row for the exemption's fail-closed boundaries, an unset Deps.DecisionRecord, a snapshot taken while the record was absent, and a new snapshot clearing an earlier snapshot's copy.
    fix: >-
      Add rows to TestJudgeSnapshot in internal/driftengine/judgesnapshot_test.go where an added entry is named as a change in three setups.
      In the first, the row clears DecisionRecord on the fixture's Deps.
      In the second, the record is absent at the snapshot and created with an entry afterwards.
      In the third, a first snapshot holds the record, the record is removed, a second snapshot is taken, and the record then returns with an added entry.
    origin: lens:code-tests
---

### [BLOCKING] The refusal rejects a valid unkeyed addition

**Location:** internal/driftengine/expectation.go:297

**Issue:** A baseline case whose every line is keyed can still gain an unkeyed line in the current capture.
`listLines` in `internal/driftengine/diff.go` numbers each unkeyed line by its position among the case's unkeyed lines, so the first such line gets `Line: 1`.
`DiffCaptures` emits a `CellAddedLine` for it, and `coversLine` covers it with a range of `1`.
A `keys` entry never covers an unkeyed cell, so a range is the only expectation that can name it, and the new branch refuses every range on that case.
`_lym/landing/summary.md` itself says that refusing ranges that hold no baseline position would refuse such a valid expectation, yet the condition refuses exactly that case.
The refusal is not needed for the goal: the issue's second option, telling Drift-Expect to write key-based entries, is built by the new `seats.go` sentence alone.
Without the refusal, the test lens's finding about the condition's unpinned halves no longer applies.

### [MEDIUM] Sealed expectations from before the change stop parsing

**Location:** internal/driftengine/check.go:244

**Issue:** `sealedAndAmended` re-parses the sealed `expected.yaml` with `ParseExpectation` on every check.
A run whose expectation was sealed before this change with a range over a fully keyed case, which is the state of the run behind issue 141, now returns `drift: the sealed expectation does not parse` before any amendment merges.
An amend adds entries and never rewrites the sealed bytes, so the issue's documented workaround stops working, and the error names no way forward.

### [MEDIUM] The exemption passes judge edits the parent's verb never makes

**Location:** internal/driftengine/judgesnapshot.go:331

**Issue:** `IdentityText` strips every well-formed carry-over block and every added entry from both the copy and the current record, and `CheckAddedEntriesSubset(now, before)` checks only that the copy's added entries survive.
A judge that rewrites or drops an existing carry-over block therefore passes, and the bound in `doc.go` names only blocks the judge writes.
A judge that appends a well-formed `### Added after Discussion (parent, …)` entry and leaves it uncommitted also passes, although the parent's verb always commits.
Such an entry carries a parent decision's authority with later rows, among them the darn gate reprompt that tells the writer to leave findings a decision forbids fixing unfixed.

### [MEDIUM] Two forms of decision add still halt the judge

**Location:** internal/driftengine/judgesnapshot.go:270

**Issue:** The issue expects the verb's commits to be left out of the judge's check.
With `--amends-plan`, `beginAmendment` and `commitAmendment` in `internal/loomcli/decision.go` also write and commit `_lym/loom/amendment.yaml`.
That file lies in no `loomRecordsDirs` entry, and the tapestry recipe runs Drift-Verify after plan approval, so the run still halts on it.
With a PATTERN entry, `AppendDecisions` inserts or withdraws a bullet in the record's PATTERN section.
`IdentityText` keeps those bullets, so the identity texts differ and the record is named as changed.

### [LOW] The Deps.DecisionRecord wiring is untested

**Location:** internal/loomcli/driftwiring.go:249

**Issue:** `export_test.go` builds `driftDeps` with an empty `driftTables{}`, and no loomcli test reads `DecisionRecord`.

### [LOW] The exemption's fail-closed boundaries have no rows

**Location:** internal/driftengine/judgesnapshot_test.go:323

**Issue:** Every new row inherits `DecisionRecord` from `newJudgeFixture` and holds the record at snapshot time.
No row shows that an unset field, an absent copy or a cleared earlier copy keeps an added entry named.

## Rejected

- Test lens, MEDIUM on the refusal condition's unpinned halves (`unkeyed == 0`, `len(occurrences) > 0`): merged into F1, whose fix removes the condition.
- Affected-set lens, LOW on the narrow bound wording in `doc.go`: merged into F3, whose fix rewrites the bound.
- Correctness lens finding 1 and affected-set finding 1 describe the same sealed-expectation defect: merged into F2.
- Assistant F1 at BLOCKING: kept at MEDIUM as F3, because the package doc states part of the bound openly and the issue licenses an exclusion decided by content; the undisclosed and uncommitted parts are fixed under F3.
- Assistant F3 (PATTERN-entry form): merged into F4 with the scope lens's `--amends-plan` finding.
- Assistant F4 (read-error rows): read errors propagate unchanged through `Changes`, the same path every existing snapshot read error takes; the absent-copy part is merged into F6.
- Test lens, LOW on an unset `DecisionRecord`, and LOW on the cleared earlier copy: merged into F6.
