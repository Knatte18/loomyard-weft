---
round: 3
ledger:
  - key: assistant-report-path-outside-sandbox
    rounds: [1, 2]
    status: resolved
  - key: retry-reads-stale-marker-and-report
    rounds: [1]
    status: resolved
  - key: codex-gets-no-heddle-instructions
    rounds: [1, 2]
    status: resolved
  - key: tui-settings-keys-unnamed
    rounds: [1]
    status: resolved
  - key: auth-json-link-survival-untested
    rounds: [1]
    status: resolved
  - key: auto-trust-plus-hook-bypass-unbounded
    rounds: [1, 2]
    status: resolved
  - key: codex-fan-before-step-4
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: interrupted-retry-chair-and-assistant-not-probed
    rounds: [2]
    status: resolved
  - key: auth-json-regular-file-after-login
    rounds: [2]
    status: resolved
  - key: assist-timeout-not-below-chair-timeout
    rounds: [2]
    status: resolved
  - key: no-marker-text-for-failed-seat-gate
    rounds: [2]
    status: resolved
  - key: d12-misstates-codex-file-render-with-no-entry
    rounds: [2]
    status: resolved
  - key: support-log-turn4-contradicts-d6-start-retry
    rounds: [2]
    status: resolved
  - key: hook-warn-misses-main-checkout-config
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: writable-roots-config-answerable-from-source
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: pre-assistant-record-reading-unstated
    rounds: [3]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: marker-location-and-tracked-status-unstated
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: support-log-turn3-stale-sandbox-reasoning
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 3 closed the cross-round backlog: every key open after round 2 is absent from the round 3 review and is resolved.
That covers the BLOCKING report-path key, the hook-bypass key (rules refusal now checks the right directory), the heddle-instructions key (D12 decides it), the weft-root bound, and the interrupted-retry probe.

Five keys are new in round 3:

- `hook-warn-misses-main-checkout-config` (F1, MEDIUM design): the Warn for committed hooks names the worktree's `.codex/config.toml`, but Codex reads a linked worktree's `hooks` table from the main checkout.
- `writable-roots-config-answerable-from-source` (F2, LOW design): the config half of the `writable_roots` question is settled by the Codex source; only enforcement on the linked path stays for the build.
- `pre-assistant-record-reading-unstated` (F3, LOW design): a record without the assistant fields has no stated reading.
- `marker-location-and-tracked-status-unstated` (F4, LOW consistency): marker location and whether markers are tracked.
- `support-log-turn3-stale-sandbox-reasoning` (F5, LOW consistency): remnant of the old report-path reasoning in Turn 3.

F1 is mapped to a new key, not to the earlier hook-bypass key, because that key's gap (rules) is closed and F1 is a different file in the bound.
