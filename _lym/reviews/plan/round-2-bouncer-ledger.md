---
round: 2
ledger:
  - key: codex-strand-no-delivery
    rounds: [1]
    status: resolved
  - key: orchengine-reminder-docs-missed
    rounds: [1]
    status: resolved
  - key: interrupt-target-record-failure-after-key
    rounds: [1]
    status: resolved
  - key: noturn-interruptguard-branch-untested
    rounds: [1, 2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: llm-smoke-waiter-lifetime
    rounds: [1]
    status: resolved
  - key: unreadable-pause-refusal-unwrapped
    rounds: [1]
    status: resolved
  - key: extends-on-rewritten-refs
    rounds: [1]
    status: resolved
  - key: hookread-resign-arrow
    rounds: [1]
    status: resolved
  - key: hookreadexit-test-duplicates-cli
    rounds: [1]
    status: resolved
  - key: no-hooks-refusal-typed-notice-claim
    rounds: [1]
    status: resolved
  - key: hookwait-refusal-external-way-forward
    rounds: [1]
    status: resolved
  - key: capabilities-header-stale
    rounds: [1]
    status: resolved
  - key: prepare-test-header-stale
    rounds: [1]
    status: resolved
  - key: journal-close-test-row-panics
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: llm-smoke-bell-pane-assertion
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: interrupt-refusals-indistinguishable
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: bound-sentence-wrong-branch
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 13 findings, one BLOCKING (the decision gap on strands with no provider tag).
Round 2 re-raised none of them except the card 7 test-reach gap, now narrowed to the guarded branch's too-short and no-box conditions (F4, scope LOW).
The other round 1 keys are resolved.

Round 2 raised four new keys.
Two are design MEDIUMs, first raised this round, and Go carries them into Open risks: card 15's journal-closing test row panics instead of returning an error (F1), and card 18's bell assertions read a pane where the line wraps (F2).
The others are card 15's three refusals that a caller cannot tell apart (F3, design LOW, a contract change) and card 8's Bound sentence naming the wrong branch for hand-added strands (F5, NIT).
