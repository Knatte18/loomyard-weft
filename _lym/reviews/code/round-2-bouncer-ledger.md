---
round: 2
ledger:
  - key: resume-startup-satisfied-by-old-events
    rounds: [1]
    status: resolved
  - key: resume-clears-skills-changes-session-scope
    rounds: [1]
    status: resolved
  - key: resume-manifest-copy-not-refreshed
    rounds: [1]
    status: resolved
  - key: old-strand-removed-before-record-names-new
    rounds: [1]
    status: resolved
  - key: step-stop-skips-attach-age-guard
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resume-discards-unconfirmed-holder-warning
    rounds: [1]
    status: resolved
  - key: leftover-runs-not-reconciled-on-restep
    rounds: [1, 2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: resume-paths-untested
    rounds: [1]
    status: resolved
  - key: panic-orphan-hook-untested
    rounds: [1]
    status: resolved
  - key: failed-save-leaves-state-naming-removed-strand
    rounds: [1]
    status: resolved
  - key: resume-overwrites-original-prompt
    rounds: [1]
    status: resolved
  - key: step-stop-reports-finished-run-as-stopped
    rounds: [1]
    status: resolved
  - key: summary-misstates-attach-untracked-rule
    rounds: [1]
    status: resolved
  - key: orphaned-hook-wiring-untested
    rounds: [1]
    status: resolved
  - key: empty-note-skip-no-row
    rounds: [1]
    status: resolved
---

Round 1's fifteen findings narrowed to two in round 2; thirteen are resolved.
`leftover-runs-not-reconciled-on-restep` (round 2 F1) stays open as a MEDIUM scope finding:
the re-step reconcile works for a live loop's interrupted child but not for the dead-loop stop, which writes no interrupted step record.
`step-stop-skips-attach-age-guard` (round 2 F2) stays open at LOW design:
the Held guard landed, but Claude's holder lookup skips an unreadable or undecodable registry entry and reports no holder.
Both fixes name the exact site and a local correction, so both are mechanical.
