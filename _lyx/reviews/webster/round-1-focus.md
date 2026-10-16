---
round: 1
focus:
  - "Card 5 anchor retirement: check the read-time remap of `below-parent` and `top` and the write-time refusal in `internal/reedengine` and `render`; settle whether a persisted strand or config carrying a retired anchor still loads, and whether `render` still imports only `encoding/json`."
  - "Card 7 `move-shell-window-strands`: check the reconcile-head move that breaks a pane out of the shell window; settle whether it is idempotent, handles a missing or already-moved pane, and has test coverage for each branch."
  - "Cards 1-3 deletions (watch loop, shell-window stack, zoom and pane buttons): settle whether anything outside the deleted code still depends on it, such as the daemon's name repair, the `window-resized` unpin on boot and attach, the Alt+z unbind in `internal/vscode/keybindings.go`, and the integration tests."
  - "Card 4 `RetiredKeys`: check that `collapsed_rows`, `min_full_rows` and `selvage` in an existing config are tolerated as retired and not rejected as unknown."
  - "Card 6 shuttle `--anchor`: settle whether `own-window|hidden` is accepted and the rest refused in `internal/shuttlecli/run.go` and `internal/shuttleengine/spec.go`, and whether the webstercli callers of the anchor still pass a valid value."
  - "Test economy across the large test deletions and additions: check that behavior kept by the diff still has a covering test, and that no new test duplicates an existing one."
  - "Per-card mechanical check: confirm the Delete and Rename groups' checks ran against every target, and that the plan's final `verify:` greps for `AnchorBelowParent`, `AnchorTop`, `below-parent` and `#673` hold."
---

Seed focus from the plan's card index and the diff stat.
No round has been judged yet.
