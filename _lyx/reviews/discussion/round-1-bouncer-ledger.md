---
round: 1
ledger:
  - key: d4-no-route-after-landing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-stop-removes-own-orch
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-restore-on-failed-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-prime-root-field-dropped
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-whole-file-replace-deletes-rules
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d3-no-pull-first-write
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-unchanged-content-push-undecided
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d3-prime-only-bound-misstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: d3-temp-file-second-bolt-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d2-heading-level-rule-no-carrier
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, none carried from an earlier ledger.
Two BLOCKING design findings sit in D4, the post-landing move: it has no route that reaches the orch on a PR-Gate approve landing, and its first step stops the orch's own session.
D3's write path has the next cluster: a restore on a failed commit that configcli does not do, no pull-first write, an undecided push on unchanged content, a prime-only bound that is a working-directory check, and a temporary file outside any Bolt exclude.
D1 drops a field the watcher also uses as the prime root, and D5 does not tell the orch that the verb replaces the whole file.
