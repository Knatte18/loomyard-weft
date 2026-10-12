---
round: 1
ledger:
  - key: d3-created-dir-bound-misses-custom-and-member-glyph
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-refusal-spec-step-interrupted-row-not-in-scope
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: d5-bound-leaves-resume-and-reseed-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d5-no-restep-rationale-covers-decode-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-criteria-miss-d2-d7-and-test-unstated-behaviour
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d5-root-cause-sentence-belongs-in-support-log
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1, first ledger.
Two blocking gaps: D3's created-directory bound ignores Custom groups and member-glyph Create targets, and D5's changed shed stop lacks its `refusal-spec.md` row in scope.
D5's way forward also dead-ends at status-missing and leaves resume behavior unstated.
Three LOW findings concern D5's no-re-step rationale, acceptance-criteria coverage for D2 and D7, and a root-cause sentence that belongs in the support log.
