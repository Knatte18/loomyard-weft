---
round: 1
ledger:
  - key: allowlist-keeps-pure-shedcheck-in-production
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landed-rule-widens-board-entry-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: foreign-board-refusal-lost-its-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pushanchored-kept-as-tested-test-copy
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: shed-recipe-spec-promises-shedbuild-load
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fabricengine-doc-names-gone-snapshot-reader
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: snapshotwarpsha-tests-test-test-code
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: checkaddeddecisionskept-doc-names-test-helper
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-names-test-only-planglyph-wrappers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-keep-markers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hibernate-terminal-phases-hand-listed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-stdout-stderr-split-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: generic-test-support-functions-not-roots
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-exemptions-differ-from-pattern-text
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-names-keep-removed-seam-names
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: newpreflight-test-names-test-helper
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: benchmark-doc-names-deleted-runcli
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: prompts-allowlist-reason-partial-consumers
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: tmuxprocessname-comment-wrong-host
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 19 findings, mapped one to one onto the keys above in review order (F1 to F19).
The gating ones are the BLOCKING `allowlist-keeps-pure-shedcheck-in-production` (F1) and the MEDIUM design findings
`foreign-board-refusal-lost-its-test` (F3) and `pushanchored-kept-as-tested-test-copy` (F4).
`landed-rule-widens-board-entry-unrecorded` (F2) is deferred to the run's parent.
No earlier ledger exists, so nothing carries forward.
