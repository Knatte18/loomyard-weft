```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fix carried out as proposed (F1, F2, F3).

**LOW: F1, the walk-only reinterpretation leaves one reconcile path able to commit the broken file**
- **Changed text:** the `defaults.arras` sentence added to `internal/runprofile/doc.go`, and the `profiles` row's new `Reinterpretations` entry in `internal/configreg/configreg.go`.
- **Case:** a hub with `defaults.arras: light` runs `lym config reconcile --layer default --apply` or `lym fabric reconcile` on the new binary before any walk has run. `ReconcileHubWideAt` drops `light.arras`, keeps the default, and commits. No reinterpretation runs on these paths.
- **Evidence:**
  - `internal/configcli/hubwide.go` (`reconcileDefaultLayer`) and `internal/fabriccli/fabric.go` (`runReconcile`) call `ReconcileHubWideAt` only.
  - My grep found no `ReconcileBeforeLoad` or `hubreconcile.Ensure` call in `configcli`, nor on the `fabric reconcile` verb; the `fabriccli` hits are in clone and the merge verbs.
- **Effect:** `runprofile.Load` fails until some verb runs the walk. The walk then heals the file, because the marker is still absent. So the window is temporary.
- **Note:** `PATTERN-config-layers` restricts reinterpretations to the walk, so this may only merit a mention in the fixer-report.

**LOW: F1, "one-time" does not cover a file that returns to the pre-drop state after the marker is written**
- **Changed text:** `ArrasDefaultReinterpretationID` and its registration, and the new "one-time reinterpretation drops it" sentence in `_lym/landing/summary.md`.
- **Case:**
  1. A new binary walks the hub and writes the board marker.
  2. An older binary, on another machine or after a revert, walks it and adds the arras blocks back.
  3. The operator sets `defaults.arras: light` again.
  4. The new binary walks again. The follow removes the untouched `light.arras` and keeps `light`; the marker suppresses the rewrite, so `Load` refuses the file.
- **Evidence:**
  - `applyReinterpretations` skips a change whose marker exists (`internal/hubreconcile/reinterpret.go:55-58`).
  - `PATTERN-config-layers` describes the older-then-newer binary add and remove cycle.
- **Related:** a revert also does not restore the operator's dropped `light`; the follow would add `defaults.arras: solid`. The "two-way" door therefore loses that one value, which is recoverable only from board history.
- **Reach:** narrow, since it needs an operator edit under an older binary.

F2 and F3: no findings. `TestLoad`'s template row still fails if an arras default returns.

Tool calls:
1. Read `internal/hubgeom/hubgeom.go` and grepped for reconcile callers: my context lacked which function triggers the walk.
2. Grepped `ReconcileBeforeLoad` and `hubreconcile.Ensure` call sites: my context lacked whether `config reconcile` or `fabric reconcile` run the walk first.
