---
round: 1
ledger:
  - key: continuesitem-excludes-too-many-hash-and-angle-lines
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: html-and-blockquote-exclusions-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-blast-radius-too-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: thematic-break-joins-bullet
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: splice-roundtrip-unindented-continuation-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: spec-sentences-split-whole-word-reasoning
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: sectionbullets-test-doc-lags-rows
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round.
The open findings center on `continuesItem` in `internal/discussionparser/decisionids.go`:
which unindented lines end a bullet, and which of those endings the tests pin.
The rest concern the change description's blast radius, the placement of the new spec sentences and a test doc comment.
