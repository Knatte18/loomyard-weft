---
verdict: APPROVED
---

No findings.

Range: `git diff $(git merge-base main HEAD)..HEAD`, since `product.parent` is `main` and no rework round exists.

Settled against the focus directive:

- Card 5: `render.Anchor.UnmarshalJSON` reads `below-parent` and `top` as `own-window` and `render` imports only `encoding/json` and `fmt`.
  `validateDisplay` refuses both with a way forward, and `AddStrandUnless` runs it before any tmux boot.
  Decode and refusal are covered by `render/types_test.go` and `strand_test.go`.
- Card 7: `listReconcilePanesLocked` is reached from launch, resume and the shared apply tail.
  It is idempotent, since a moved pane leaves the shell window and the next listing finds none.
  `listreconcile_test.go` covers a failed break, a malformed answer, a failed marker, a failed first listing and a failed relist.
- Cards 1-3: the name repair stays on `listStrandAndWindowPanes`, boot and attach still unpin `window-resized`, and `bindings.go` still unbinds Alt+z.
  Only the seed entry for Alt+z was dropped from `internal/vscode/keybindings.go`.
- Card 4: `collapsed_rows`, `min_full_rows` and `selvage` sit in the `RetiredKeys` map in `internal/reedengine/config.go` as `CategoryNone`.
- Card 6: `lyx shuttle run --anchor` refuses everything but `own-window` and `hidden`.
  The webstercli callers carry no anchor of their own.
- The plan's final `verify:` greps for `AnchorBelowParent`, `AnchorTop`, `below-parent` and `#673` all come back empty.
- The tests of `reedengine`, `render`, `shuttlecli`, `shuttleengine`, `webstercli` and `vscode` pass under `lyx gate test`.
