---
round: 2
ledger:
  - key: assistant-report-path-outside-sandbox
    rounds: [1, 2]
    status: open
    class: consistency
    severity: BLOCKING
    fix: design-wrong
  - key: retry-reads-stale-marker-and-report
    rounds: [1]
    status: resolved
  - key: codex-gets-no-heddle-instructions
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tui-settings-keys-unnamed
    rounds: [1]
    status: resolved
  - key: auth-json-link-survival-untested
    rounds: [1]
    status: resolved
  - key: auto-trust-plus-hook-bypass-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: codex-fan-before-step-4
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: chair-wait-bound-and-start-retry-overstated
    rounds: [1]
    status: resolved
  - key: d1-no-channel-contradicts-d7-pane-mail
    rounds: [1]
    status: resolved
  - key: assist-stencil-parent-directive-unstated
    rounds: [1]
    status: resolved
  - key: skills-sync-open-risk-is-writers-choice
    rounds: [1]
    status: resolved
  - key: weft-writable-root-unbounded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: interrupted-retry-chair-and-assistant-not-probed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: auth-json-regular-file-after-login
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: assist-timeout-not-below-chair-timeout
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: no-marker-text-for-failed-seat-gate
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d12-misstates-codex-file-render-with-no-entry
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: support-log-turn4-contradicts-d6-start-retry
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 2 reviewed the record after the fixer's round-1 pass.
Round 1's retry-marker, `[tui]` keys, `auth.json` link research, pane-mail contradiction, parent-directive and skills-sync keys no longer appear and are resolved.
`chair-wait-bound-and-start-retry-overstated` is resolved in the decision record; its remnant in the support log's Turn 4 is the new NIT key `support-log-turn4-contradicts-d6-start-retry`.

Reopened keys:

- `assistant-report-path-outside-sandbox` is now F1, BLOCKING.
  The parent's weft-writable-root entry replaced the premise; D2, D6, Constraints, Open risks and the D2 acceptance criteria must follow it.
- `auto-trust-plus-hook-bypass-unbounded` is now F4: the committed `.codex/rules` allow rules that automatic trust loads.
- `codex-gets-no-heddle-instructions` is now F5: no `claude-file` entry reaches the Codex seat by default.
- `codex-fan-before-step-4` is now F6: the parent settled it, so the Open risks line and Q4 must go.

New in round 2: the unbounded weft writable root (F2), the retry probe built from the bare round (F3), a regular `auth.json` after a login (F7), the assist timeout not held below the chair's (F8), the missing failed-gate marker text (F9), the D12 render claim (F10), and the Turn 4 contradiction (F11).
