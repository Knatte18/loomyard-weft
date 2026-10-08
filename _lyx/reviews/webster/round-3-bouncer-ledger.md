---
round: 3
ledger:
  - key: comment-fixed-column-wraps
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: BLOCKING
  - key: redundant-package-target-unit-as-path
    rounds: [1]
    status: resolved
  - key: spec-paragraph-multi-sentence-lines
    rounds: [1]
    status: resolved
  - key: duplicate-findings-repeated-target
    rounds: [1]
    status: resolved
  - key: interface-method-func-prefix-detection
    rounds: [1]
    status: resolved
  - key: scan-directory-skips-untested
    rounds: [1]
    status: resolved
  - key: package-overview-omits-plangatepass
    rounds: [1]
    status: resolved
  - key: test-row-pins-unreachable-input
    rounds: [1]
    status: resolved
  - key: repeated-plan-gate-assertion
    rounds: [1]
    status: resolved
  - key: edittarget-doc-comment-unparseable
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: glyph-rejected-pass-list-incomplete
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: crosscard-test-unused-want-column
    rounds: [3]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised nine findings; round 2 found those fixed except the comment-wrap defect, which survived at five more edited sentences.
Round 3 found the same defect at three further edited sentences (F1, BLOCKING), so `comment-fixed-column-wraps` has been open three rounds running.
The pattern is that each fix covers the named sites only and the next review finds others.

Round 3 also re-opened the `editTarget` doc (F2, LOW) and added two new consistency findings: the `doc.go` list of glyph-rejected passes omits `deleteorder.go` (F3, LOW), and `TestGlyphChain_CrossCard` has an unused `want` column (F4, NIT).
No design, scope or decision finding is open.
