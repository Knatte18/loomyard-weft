```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as proposed, F12 and F13.

**Finding 1 — MEDIUM**
- Changed text: the flagged line in `internal/gateslot/reinterpret.go` (`RetireHardenLiveCopies`), which ends "then remove gates.harden-live-copies with lym config reconcile". The summary bullet says the same: "set `harden-live` to one tier, then remove the key".
- Summary: the way forward the flag names cannot be carried out, because the rewrite never runs a second time.
- Evidence:
  - `applyReinterpretations` (`internal/hubreconcile/reinterpret.go`) writes the board marker after every `Rewrite` call. That includes a call that only flagged, and it skips any reinterpretation whose marker exists (`if fileExists(marker) { continue }`).
  - So in the flagged case, such as this hub's `harden-live: [0, 1]` with `harden-live-copies: [1]`, the operator sets `harden-live` to one tier as told. A later `lym config reconcile` finds the marker, skips the rewrite, and leaves `harden-live-copies` in the file.
  - The flag text therefore promises a removal that never happens. The stale key stays in the hub file, harmless but misleading.
- Concrete case: this hub's own `gate.yaml`.
- Fix directions, not applied:
  - Drop the "then remove … with lym config reconcile" clause from the flag and the summary, and tell the operator to delete the key by hand.
  - Or make the flagged case write no marker, which needs an engine change.

**F12:** no findings. `hardenshed` already imports `shedadapters`, and the exported `FocusPath` keeps the same signature. The fixer renamed every grep hit, so no caller is left on the old name.

Calls made: none. Everything I needed was already in context.
