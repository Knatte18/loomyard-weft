---
round: 2
ledger:
  - key: heddlecli-lacks-runcli
    rounds: [1]
    status: resolved
  - key: runproc-allowlist-misses-lymcwd
    rounds: [1]
    status: resolved
  - key: new-test-packages-no-testmain
    rounds: [1]
    status: resolved
  - key: boltdirname-guard-no-lym-guard-marker
    rounds: [1]
    status: resolved
  - key: run-ledger-decision-departs-d7
    rounds: [1]
    status: resolved
  - key: heddle-cli-departs-cliwire-constraint
    rounds: [1]
    status: resolved
  - key: bolt-paths-outside-fabric-use-boarddir
    rounds: [1]
    status: resolved
  - key: fastforwardpair-no-took-in
    rounds: [1]
    status: resolved
  - key: reconcile-reseed-row-unreachable
    rounds: [1]
    status: resolved
  - key: heddle-set-no-attack-surface-bound
    rounds: [1]
    status: resolved
  - key: no-board-file-guard-scans-two-files
    rounds: [1]
    status: resolved
  - key: config-load-order-unstated
    rounds: [1]
    status: resolved
  - key: drivable-worktree-refusal-no-row
    rounds: [1]
    status: resolved
  - key: restorereplaytip-message-stale
    rounds: [1]
    status: resolved
  - key: pattern-background-files-board-name
    rounds: [1]
    status: resolved
  - key: config-go-header-stale
    rounds: [1]
    status: resolved
  - key: excludewayforward-on-wrong-card
    rounds: [1]
    status: resolved
  - key: entrymerge-table-duplicates-fabricreplay
    rounds: [1]
    status: resolved
  - key: card6-import-cycle-packages-unnamed
    rounds: [1]
    status: resolved
  - key: livestate-doc-test-board-literal
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 2 raised one finding, F1 (consistency, MEDIUM): cards 4 and 8 still leave and exempt the `_board` literal in `livestate_doc_test.go`, against the parent's recorded entry.
It maps to the round 1 key `livestate-doc-test-board-literal`, now reopened at MEDIUM.
Its class is consistency, so it is not gating, and its fix is a local card edit.
Every other round 1 key went unraised in round 2 and is resolved.
