---
round: 1
ledger:
  - key: d3-seed-push-no-place-before-cloneandwire
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-anchor-refusal-catches-two-repo-weft
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-full-name-delete-reports-missing-ref-deleted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: board-branch-name-in-namespace-undecided
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-force-replace-bound-checked-too-early
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-rerun-adopts-leftover-not-replaces
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: open-risks-researchable-board-body
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d2-gitrepo-param-list-misses-upstream-and-deletes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-git-facts-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-teardown-covers-refused-push-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d9-pattern-backgrounds-list-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-omits-single-repo-flag
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: plan-writer-note-files-wrong-package
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first ledger, so no key recurs.
The gating findings centre on D3's clone sequence (seed push placement, the `.lym-anchor` refusal, the force-replace bound, rerun behaviour),
D4's remote delete by full ref name, and the undecided board branch name inside `refs/lym/heads/`.
The remaining findings are local corrections: an Open risks line, a parameter list in D2, evidence pins in D1, D3's teardown wording, D9's list, D8's flag and the plan-writer file list.
