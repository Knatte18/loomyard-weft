---
round: 2
ledger:
  - key: unitpath-reader-count-false-premise
    rounds: [1]
    status: resolved
  - key: refusal-way-forward-wrong-binding
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: language-none-skip-unbounded
    rounds: [1]
    status: resolved
  - key: d6-overstates-d2-replacement
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: external-test-handle-risk-answered
    rounds: [1]
    status: resolved
  - key: scope-items-lack-criterion
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: testdata-exemption-narrower-than-go-ignore-set
    rounds: [1]
    status: resolved
  - key: d4-orphans-member-glyph-clause
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's gating finding, the false `Glyph.UnitPath` reader count, and the two other `design` findings are resolved: round 2 confirms D4's wording, D3's bounded exemptions and the open-risk answer.
Round 2 raised four consistency findings, none gating: D6's paraphrase of D2's condition, the Constraints citations, the missing criterion for the `UnitPath` sentence rewrite, and the member-glyph clause D4 orphans.
