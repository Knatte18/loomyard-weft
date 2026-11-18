---
verdict: BLOCKING
findings:
  - id: F7
    severity: BLOCKING
    class: consistency
    origin: handler
    location: internal/driftengine/doc.go:276
    summary: The driftengine package doc states a loomcli behavior, that lym loom decision add refuses its --amends-plan and PATTERN-change forms while a judge session is open, which PATTERN-code-conventions forbids since a package doc.go documents only that package.
    fix: In internal/driftengine/doc.go, delete the two lines reading "The verb's `--amends-plan` and PATTERN-change forms write outside that exemption," and "so the verb refuses them while the scope record's judge session is open." If a bound is wanted in their place, write one line stating only driftengine's behavior, such as that a change beside the decision record, the plan amendment record or the record's PATTERN changes section, is not exempt and is named as a change. Leave the refusal's description in internal/loomcli/doc.go, which already carries it.
  - id: F8
    severity: BLOCKING
    class: consistency
    origin: assist:assist
    location: internal/loomcli/driftwiring_test.go:64
    summary: TestDriftDeps_CarryTheDecisionRecord is a contained candidate in the redundancy report that pins what its covering test never asserts, so PATTERN-test-speed requires it to carry a testtiming keep marker, and it carries none.
    fix: In internal/loomcli/driftwiring_test.go, add the line `//testtiming:keep pins that driftDeps copies the agent tables' decision record path into Deps.DecisionRecord, which TestDriftDeps_RunWhereTheStepsDirResolves never asserts` directly above `func TestDriftDeps_CarryTheDecisionRecord`, below its doc comment. Change nothing else in the test.
prior:
  - id: F1
    status: resolved
    answer: ParseExpectation in internal/driftengine/expectation.go is unchanged against the merge base and accepts a lines range on a fully keyed baseline case; the RangeOverFullyKeyedBaselineCase row of TestParseExpectation pins the acceptance, and no refusal remains.
  - id: F2
    status: resolved
    answer: sealedAndAmended re-parses the sealed bytes and each amendment through the same ParseExpectation, which no longer refuses such a range, so an expectation sealed with one parses and the amend workaround clears it.
  - id: F3
    status: resolved
    answer: appendedDecisionRecord now refuses the exemption when the checkout lists the record as changed or untracked and when CheckCarryOversNotLost reports a dropped or changed block; rows for an uncommitted entry and an edited block are named, and the doc.go bound states what still goes unseen.
  - id: F4
    status: resolved
    answer: decisionVerb refuses --amends-plan and every PATTERN form, withdrawals included, while the scope record's judge session is open, naming when to retry; TestDecisionVerb_Refusals and TestDecisionVerb_AmendsPlan reach both refusals. The refusal is justified, since _lym/loom/amendment.yaml lies outside the loom records dirs and the PATTERN bullet survives IdentityText, so both would halt the judge.
  - id: F5
    status: resolved
    answer: TestDriftDeps_CarryTheDecisionRecord asserts that driftDeps copies the decision record path from the agent tables into Deps.DecisionRecord.
  - id: F6
    status: resolved
    answer: TestJudgeSnapshot gains named rows for an unset DecisionRecord, a record absent at the snapshot, and a later snapshot clearing an earlier copy; the last row would pass as exempt if the copy were not cleared, so it pins the clearing.
---

### [BLOCKING] driftengine's package doc describes the loomcli verb's refusal

**Location:** internal/driftengine/doc.go:276

**Issue:** The F4 fix added two lines to the judge-snapshot section of driftengine's `doc.go` that state what `lym loom decision add` in `internal/loomcli` refuses and when.
`PATTERN-code-conventions` rules that a package's `doc.go` documents only that package.
The refusal is loomcli's behavior; `internal/loomcli/doc.go` already states it in the same diff, so the driftengine lines also go stale silently if the verb's refusal changes.

### [BLOCKING] The kept wiring test lacks its testtiming keep marker

**Location:** internal/loomcli/driftwiring_test.go:64

**Issue:** The redundancy report lists `TestDriftDeps_CarryTheDecisionRecord` as contained by `TestDriftDeps_RunWhereTheStepsDirResolves`.
That covering test builds its Deps through `DriftDepsAt` with empty agent tables and asserts the capture and compare directories and environments, never `Deps.DecisionRecord`.
The candidate pins an assertion coverage cannot see, so it is kept, and `pattern/PATTERN-test-speed.md` requires a kept candidate to carry `//testtiming:keep <reason>` directly above its `func Test…` line.

## Rejected

- Assistant F8, the record path built with a `_lym` literal in `TestDriftDeps_CarryTheDecisionRecord`: the test passes an opaque path through `driftDeps` and checks it comes back unchanged, so it places no file in lym's geometry, which is what `PATTERN-lym-files` governs; the path's shape plays no part in the assertion.

## Notes outside this round's reach

The incremental-round rule allows only BLOCKING findings, so these observations are recorded here without a finding.

- The `DecisionEntryDroppedIsNamed` row of `TestJudgeSnapshot` sets `f.records.Changed`, so `appendedDecisionRecord` returns at its uncommitted-write check and never reaches `CheckAddedEntriesSubset`. No row exercises a committed drop or rewrite of an added entry.
- `driftJudgeSessionOpen` in `internal/loomcli/driftwiring.go` has no test of its own; the verb's tests replace it with a fake.
- The verb's judge-session check runs under the decision lock, but `openJudgeSession` does not take that lock. A PATTERN or `--amends-plan` call that passes the check just before the snapshot can still commit inside the session and halt the judge.
