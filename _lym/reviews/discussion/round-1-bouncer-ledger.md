---
round: 1
ledger:
  - key: holder-gone-after-read-never-swept
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: parent-question-2026-10-27-wait
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: chase-clock-not-rebuilt-after-restart
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-shuttle-send-misread-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reply-to-forwarded-chase-unread
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: program-trust-gap-orch-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: beam-question-and-hold-notice-diverge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hold-notice-plain-post-leaves-mark-owed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: retention-drops-owed-read-messages
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: crash-between-chase-post-and-chased-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: external-envelope-in-chase-body
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: never-read-bound-wrong-for-hook-strands
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: log-entry-lacks-reply-link
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d5-cites-wrong-decision-for-chase
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cycle-constraint-lacks-evidence
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: scope-carries-deliberation
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists, and no key recurs.

The gating core is the D10 chase mechanism: a holder that dies after reading is never swept (F1), the frozen clock cannot be rebuilt after a restart (F3), a reply to a forwarded chase lands unread (F5), and a plain-post answer to a hold notice leaves the marked question owed (F8).
Around it sit attack-surface gaps in D2 (program trust) and D8 (shuttle send replacement), and a D3 beam-routing divergence.

F2 is a parent question about the operator's 2026-10-27 wait and is not for the fixer.
The LOW and NIT findings are local corrections to D5, D10, D11, D12, Constraints and Scope.
