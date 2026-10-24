---
round: 1
ledger:
  - key: overview-ide-entry-stale-attach-keybinding
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: keybindings-test-comment-names-other-symbol
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two consistency findings.
F1 (`overview-ide-entry-stale-attach-keybinding`) is the `docs/overview.md` ide entry still describing the pre-change keybinding block.
F2 (`keybindings-test-comment-names-other-symbol`) is the `TestKeybindingsBlock` doc comment naming `WriteConfig`.
Neither is gating.
