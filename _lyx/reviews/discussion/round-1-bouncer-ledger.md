---
round: 1
ledger:
  - key: d5-watched-window-bound-unverified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: buildopencmd-comment-only-in-optional-notes
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: ac4-unobservable-session-id-ordering
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings.
The gating one concerns D5: its bound that the bypass session runs in a window an operator watches has no basis in the open path, and a session driven through `lyx shuttle send` goes unnamed.
The other two are local: the docs list in Scope omits two doc comments, and AC4 names an ordering no public surface can observe.
