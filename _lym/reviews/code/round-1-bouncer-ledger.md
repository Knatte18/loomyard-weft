---
round: 1
ledger:
  - key: census-ignores-sibling-real-root-scratch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: doc-go-remove-paragraph-contradicts-code
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: cleanup-anchored-scratch-path-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: anchor-change-exclude-drop-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: remove-legacy-root-link-at-subpath-anchor-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: foreign-root-entry-at-subpath-anchor-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: adopt-and-leave-rows-only-at-anchor-dot
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: plan-spec-names-repository-root-scratch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round, no prior ledger.
F1 is the one gating finding: the sibling census in `scratchLinkedInSibling` counts only a link, so a sibling's real root `.scratch` loses its exclude line when another pair retires its root link.
F2 and F8 are stale prose in `doc.go` and the plan spec.
F3 to F7 are missing tests for subpath-anchor paths the diff changes.
Each fix text names the exact change, so all eight are mechanical.
