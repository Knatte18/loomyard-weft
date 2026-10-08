---
round: 1
ledger:
  - key: edit-directive-cards-miss-handbuilt-stencil-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: pair-call-unnormalized-worktree-path
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: headcontains-test-repeats-stencilsource-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: card8-new-loomcli-test-file-duplicate-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: configengine-scratchdir-bundled-in-hubreconcile
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: decodeleafvalues-unused-path-param
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round; no prior ledger existed.
Two MEDIUM design findings gate: the edit-directive cards (17, 19) omit tests that hand-build stencils or values maps, and the pair call (cards 5, 6, 9) compares unnormalized worktree paths.
Four LOW findings remain open: a redundant `HeadContains` test (F3), card 8's new test file and duplicate row (F4), `ScratchDir` bundled into the hubreconcile card (F5), and an unused `decodeLeafValues` parameter (F6).
