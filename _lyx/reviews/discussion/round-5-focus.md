---
round: 5
focus:
  - "Zoom lifecycle: `_lyx/discussion/decision-record.md` D1 makes a zoomed pane the way to show one strand, and D4, D8 and D9 light buttons from zoom state. Reed's `window-resized` hook in `internal/reedengine/windowsize.go` (`resizePinHookArgvs`, `resize-pane -y`) and the `select-layout` calls in `applyLayoutLockedOpts` (`internal/reedengine/apply.go`) and `AttachArgv` (`internal/reedengine/attach.go`) can unzoom a window under tmux. Settle whether the record has a Decision, with a bound and an acceptance criterion, stating what a terminal resize, a re-attach, a spawn and a layout apply do to a zoomed window, and what layout the next unzoom shows."
  - "D7 pin floor and border-status mapping: check where the one-content-row floor on the row-0 pin is applied (the `window-resized` hook only, or also the apply path that issues `select-layout`), and whether the acceptance criterion names the moment it is checked. Check that the title-row subtraction keys on a `pane-border-status` readback of exactly `top`, with `off`, `bottom`, empty and a failed readback all meaning no row-0 title row."
  - "D8 run-shell quoting: `run-shell` format-expands its whole command at key press. Check that the D8 Bound states how a `#` in a pinned `lyx`, tmux or socket path is kept literal, how `#{q:client_name}` is placed relative to the single-quoted tokens built by `internal/shell`, and that D11's binary path carries the same rule."
  - "D12 and D15 residue: check that the spawn-order data the teller hands `reedengine` has a stated reader or is absent, that the booter's trigger and the revive function's skip are stated as one predicate, and that D15 states what an unreadable `keybindings.json` and a non-unique temporary file name do to `lyx ide spawn`."
---

Each entry names a place to look and a question to settle; none pre-states a verdict.
