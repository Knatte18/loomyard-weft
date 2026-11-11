---
round: 2
ledger:
  - key: relay-never-ends-after-retire-drops-last-button
    rounds: [1]
    status: resolved
  - key: forwarding-entry-without-channel-entry-never-dropped
    rounds: [1]
    status: resolved
  - key: root-project-handle-outside-limits
    rounds: [1]
    status: resolved
  - key: worktree-orch-and-orch-project-untested
    rounds: [1]
    status: resolved
  - key: failed-button-drop-after-archive-not-retried
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: pattern-daemon-wakeups-names-presence-watch
    rounds: [1]
    status: resolved
  - key: diskcheck-forwarding-comparison-untested
    rounds: [1]
    status: resolved
  - key: hook-beam-orch-own-button-untested
    rounds: [1]
    status: resolved
  - key: inbound-unreadable-button-branch-untested
    rounds: [1]
    status: resolved
---

Round 1 raised nine findings.
Round 2 raised one, F1 at LOW: the retry of a failed button drop after the archive stood is correct in code but has no test.
Its key stays open at LOW.
The other eight keys did not come back and are resolved.
