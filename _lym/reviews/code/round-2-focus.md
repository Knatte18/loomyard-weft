---
round: 2
focus:
  - "Does ParseExpectation in internal/driftengine/expectation.go accept a `lines` range on a baseline case whose every line is keyed, and one over an unkeyed line the current capture adds to such a case? Settle whether any refusal remains and whether it is justified."
  - "Does sealedAndAmended in internal/driftengine/check.go still parse an expectation sealed with a range over a fully keyed case, so the amend workaround clears it?"
  - "In appendedDecisionRecord in internal/driftengine/judgesnapshot.go, what does the decision-record exemption pass beyond what `lym loom decision add` writes and commits: an uncommitted appended entry, or an edited or dropped carry-over block? Check that the doc.go bound states what still goes unseen."
  - "For `lym loom decision add --amends-plan` and for a PATTERN-entry add, does the judge session still halt on _lym/loom/amendment.yaml or on the record's PATTERN bullet, and is each form either exempted or refused with a way forward and a test?"
  - "Does a loomcli test pin that driftDeps copies the run's decision record path into Deps.DecisionRecord, and does TestJudgeSnapshot pin the exemption's fail-closed cases: unset DecisionRecord, record absent at the snapshot, and a later snapshot clearing an earlier copy?"
---

Entries name where to look and which question to settle; severity and verdict are the reviewer's.
