---
round: 2
ledger:
  - key: d12-commit-test-inherited-startsha
    rounds: [1]
    status: resolved
  - key: d7-allowlist-refuses-nested-orch
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-reader-write-flags-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-recoveries-count-not-carried
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-new-refusal-unmarked
    rounds: [1]
    status: resolved
  - key: d9-gated-leftover-files
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-warning-no-sink-restamp-save
    rounds: [1]
    status: resolved
  - key: d1-d2-refusals-uncovered
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: friction-constraint-no-plumbing
    rounds: [1]
    status: resolved
  - key: goal-stale-tally-interview-contradiction
    rounds: [1]
    status: resolved
  - key: scope-bullets-without-criteria
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
  - key: mergeresolve-import-question-in-support-log
    rounds: [1]
    status: resolved
  - key: d11-candidates-perishable-criterion-not-red
    rounds: [1]
    status: resolved
  - key: d8-d5-splitter-and-pinned-tests
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
  - key: d7-guard-refuses-detached-push
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-refusal-spec-ways-forward
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d12-card-amended-cap-no-master-rung
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-test-claims-disagree
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d8-classifier-seam-unstated
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1 raised fourteen findings, three BLOCKING.
Round 2's fresh review did not re-raise the D12 commit baseline, the D8 BLOCKING flag bound as stated, or several consistency items, so those are resolved; the D8 and D7 areas reappear in new forms.

Round 2 adds one BLOCKING finding: D7's guard refuses the detached push child (`d7-guard-refuses-detached-push`).
It also adds MEDIUM design findings on D7's refusal-spec rows, D12's `card_amended` re-run against the cap, and D8's classifier seam, plus a MEDIUM consistency finding on D9's test claims.
Five round-1 keys stay open at MEDIUM design: D7 nested calls, D8 reader forms, D12 counters on re-begin, D9 gated leftovers and D1's refusal text.
LOW round-2 findings (F11 to F18) are not tracked as separate keys; F15 continues the splitter key and F19 the acceptance-criteria key.
