---
round: 1
ledger:
  - key: assistant-report-path-outside-sandbox
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: retry-reads-stale-marker-and-report
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: codex-gets-no-heddle-instructions
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tui-settings-keys-unnamed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: auth-json-link-survival-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: auto-trust-plus-hook-bypass-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: codex-fan-before-step-4
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: chair-wait-bound-and-start-retry-overstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d1-no-channel-contradicts-d7-pane-mail
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: assist-stencil-parent-directive-unstated
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: skills-sync-open-risk-is-writers-choice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round; there was no previous ledger.
The keys map to the review's findings in order, F1 to F11.

The blocking issue is F1: the assistant's report path resolves through the anchor's `.lym` link outside the Codex workspace-write sandbox.
F2 to F6 are gating design gaps: stale marker and report on a retry, missing heddle instructions for Codex, unnamed `[tui]` keys, an untested `auth.json` link, and an unbounded hook-trust edge.
F7 is a parent decision on a Codex subagent fan before Step 4.
F8 to F11 are LOW corrections.
