---
round: 5
ledger:
  - key: d7-border-row-subtraction-false-premise
    rounds: [1, 2]
    status: resolved
  - key: d12-booter-session-order-and-nested-up-recursion
    rounds: [1, 2, 3, 4, 5]
    status: open
    class: decision
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
    status: resolved
  - key: d7-top-pin-one-row-floor
    rounds: [3, 4]
    status: resolved
  - key: d8-runtime-format-single-quote-bound
    rounds: [3, 4]
    status: resolved
  - key: zoom-dropped-by-resize-hook-and-layout-apply
    rounds: [4, 5]
    status: open
    class: decision
    severity: LOW
  - key: d15-unreadable-keybindings-file-open
    rounds: [4]
    status: resolved
  - key: d15-temp-file-name-not-unique
    rounds: [4, 5]
    status: open
    class: decision
    severity: LOW
  - key: support-log-missing-round-3-entry
    rounds: [4]
    status: resolved
  - key: zoom-pane-order-read-while-zoomed
    rounds: [5]
    status: open
    class: design
    severity: MEDIUM
  - key: zoom-unzooming-commands-incomplete
    rounds: [5]
    status: open
    class: design
    severity: LOW
  - key: constraints-start-times-leftover
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
  - key: scope-line-narrows-session-recreation
    rounds: [5]
    status: open
    class: consistency
    severity: LOW
  - key: d12-bound-restates-predicate-without-not-live
    rounds: [5]
    status: open
    class: consistency
    severity: NIT
---

Round 5 has one gating finding: F1, a new MEDIUM design finding.
D1 reads the pane list without saying it is read on the unzoomed window, while a zoomed pane reports `pane_top` 0, so D7's row 0 and the layout string are wrong under zoom (`zoom-pane-order-read-while-zoomed`).
F2 (`zoom-unzooming-commands-incomplete`) is the LOW companion: D1's re-zoom trigger names two unzooming commands where tmux has more.

The round-4 BLOCKING zoom finding shrinks to F3 (`zoom-dropped-by-resize-hook-and-layout-apply`), a LOW decision gap about how the attach chain and the hook entries carry the bracket and the pins.
The D7 floor, the border-status mapping and D8's quoting are settled by the review's focus verdicts, so those keys and the D15 unreadable-file key are resolved.
The round-4 support-log NIT is not raised again and is resolved.

The D12 booter key recurs as F5, a LOW decision.
D15's temporary-name collision remains as F6, a LOW decision.
F4, F7 and F8 are new LOW or NIT consistency leftovers: the Constraints start-times bullet, the Scope line and the D12 Bound predicate.
