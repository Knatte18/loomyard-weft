---
round: 4
ledger:
  - key: d7-border-row-subtraction-false-premise
    rounds: [1, 2]
    status: resolved
  - key: d12-booter-session-order-and-nested-up-recursion
    rounds: [1, 2, 3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: d12-concurrent-booter-order-race
    rounds: [1]
    status: resolved
  - key: d15-jsonc-insertion-not-byte-identical
    rounds: [1]
    status: resolved
  - key: config-colors-decision-bolted-on
    rounds: [1]
    status: resolved
  - key: config-color-values-unbounded-reader-unnamed
    rounds: [1, 2]
    status: resolved
  - key: d13-claude-color-names-vs-provider-seam
    rounds: [1]
    status: resolved
  - key: pre-change-pane-criterion-outruns-d2-d14
    rounds: [1]
    status: resolved
  - key: support-log-missing-parent-change
    rounds: [1]
    status: resolved
  - key: d13-color-refusal-at-load-blast-radius
    rounds: [2]
    status: resolved
  - key: d12-lock-placement-and-cross-worktree-up
    rounds: [2, 3]
    status: resolved
  - key: d15-atomic-write-readonly-contradiction
    rounds: [2, 3]
    status: resolved
  - key: d15-marker-edge-cases
    rounds: [2]
    status: resolved
  - key: d7-top-pin-row0-source-undefined
    rounds: [3]
    status: resolved
  - key: d7-top-pin-ignores-refused-border-status
    rounds: [3, 4]
    status: open
    class: design
    severity: LOW
  - key: d7-top-pin-one-row-floor
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-runtime-format-single-quote-bound
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
  - key: zoom-dropped-by-resize-hook-and-layout-apply
    rounds: [4]
    status: open
    class: design
    severity: BLOCKING
  - key: d15-unreadable-keybindings-file-open
    rounds: [4]
    status: open
    class: decision
    severity: LOW
  - key: d15-temp-file-name-not-unique
    rounds: [4]
    status: open
    class: decision
    severity: NIT
  - key: support-log-missing-round-3-entry
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Round 4 settles the D7 row-0 source and the D12 lock/teller-input questions; the review's focus verdicts mark both as not defects, so those keys are resolved.
The D15 read-only contradiction is gone too (the identical-content no-op precedes the read-only check), replaced by two narrower D15 gaps: an unreadable file and a non-unique temporary name.
The D7 and D8 keys stay open with a deeper defect each: the one-row floor holds only after a resize event (F2), the refused-`border-status` mapping should key on exactly `top` (F4), and D8's Bound misses format expansion of the pinned tokens and unquoted `q:` use (F3).
F1 is new and BLOCKING: D1's zoom does not survive reed's `window-resized` pin hook or its `select-layout` apply paths.
The D12 spawn-order key (F7) is the remaining LOW consistency item; the support log lacks a round 3 entry (F8).
