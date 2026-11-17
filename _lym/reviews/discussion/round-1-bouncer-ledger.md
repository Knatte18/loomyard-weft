---
round: 1
ledger:
  - key: d6-fabric-ide-knowledge
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: board-open-subdir-resolve
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-link-allowlist-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-beam-start-binary-deferred
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-mixed-version-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: board-open-refusal-invisible
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-phone-goal
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: parent-ide-prime-setup-remainder
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d6-tracked-tasks-json-misattributed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d1-projection-no-section-order
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d4-design-file-link-may-not-exist
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-second-keypress-active-task
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d3-downgrade-replay-misstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-names-no-escape
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: scope-out-carries-history
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 was the first review: fifteen findings, none carried from an earlier ledger.

Gating: the BLOCKING fabric-boundary finding (D6 writes tasks inside `fabricengine`) and five MEDIUM design findings
(subdirectory resolution of `lym board open`, the undefined link allowlist, the undecided beam start binary,
the missing mixed-version bound for D6, the invisible refusal of the board open task).

Parent questions, not for the fixer: the phone goal against D7, and the remainder of the Was ide-prime-setup part.

Mechanical or LOW remainder: tracked `tasks.json` attribution, section order in the projection, design-file link guard,
second key press while the opener blocks, downgrade replay wording, the unnamed JSON escape, history in Scope Out.
