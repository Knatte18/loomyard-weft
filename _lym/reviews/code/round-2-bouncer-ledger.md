---
round: 2
ledger:
  - key: allowlist-keeps-pure-shedcheck-in-production
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: landed-rule-widens-board-entry-unrecorded
    rounds: [1]
    status: resolved
  - key: foreign-board-refusal-lost-its-test
    rounds: [1]
    status: resolved
  - key: pushanchored-kept-as-tested-test-copy
    rounds: [1]
    status: resolved
  - key: shed-recipe-spec-promises-shedbuild-load
    rounds: [1]
    status: resolved
  - key: fabricengine-doc-names-gone-snapshot-reader
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: snapshotwarpsha-tests-test-test-code
    rounds: [1]
    status: resolved
  - key: checkaddeddecisionskept-doc-names-test-helper
    rounds: [1]
    status: resolved
  - key: overview-names-test-only-planglyph-wrappers
    rounds: [1]
    status: resolved
  - key: contained-tests-lack-keep-markers
    rounds: [1]
    status: resolved
  - key: hibernate-terminal-phases-hand-listed
    rounds: [1]
    status: resolved
  - key: run-stdout-stderr-split-untested
    rounds: [1]
    status: resolved
  - key: generic-test-support-functions-not-roots
    rounds: [1]
    status: resolved
  - key: guard-exemptions-differ-from-pattern-text
    rounds: [1]
    status: resolved
  - key: test-names-keep-removed-seam-names
    rounds: [1]
    status: resolved
  - key: newpreflight-test-names-test-helper
    rounds: [1]
    status: resolved
  - key: benchmark-doc-names-deleted-runcli
    rounds: [1]
    status: resolved
  - key: prompts-allowlist-reason-partial-consumers
    rounds: [1]
    status: resolved
  - key: tmuxprocessname-comment-wrong-host
    rounds: [1]
    status: resolved
  - key: summary-describes-round-1-tree
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised 19 findings; round 2 raised three.
Round 2 did not re-raise 17 of the round-1 keys, so they are resolved.

The allowlist key recurs: round 1 found pure `shedcheck` code kept in production, and round 2 finds two more entries,
`battenrecipe.RecipeEngines` and `fabricvocab.FailsBareCheck`, that read no own-package state and write no record.
It stays BLOCKING and gating.

`fabricengine-doc-names-gone-snapshot-reader` reopens at LOW: the package doc still describes a production snapshot reader.

`summary-describes-round-1-tree` is new in round 2 (MEDIUM, consistency).
The run's parent owns `_lym/landing/summary.md`, so the fixer defers it.
