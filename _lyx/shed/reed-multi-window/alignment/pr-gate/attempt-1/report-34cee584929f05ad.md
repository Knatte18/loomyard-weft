---
verdict: APPROVED
---

PR 599 at `1144cd2f4` against `main` matches the board entry's goal and the decision record's D1–D12.

- reed: `own-window` is a supported placement (`render` accepts it with `Window` and `WindowSize`), with launch through the strand chokepoint, session-wide presence, groups up to `window_share_max` split by `window_split`, removal that moves a viewer to window 0, resume, and the status-bar button marked unseen (cards 1–7).
- shuttle: `own-window` is the default in `Spec.validate` and in `lyx shuttle run --anchor`; the orch names `below-parent` (D10).
  The stop paths remove dead own-window strands, and `Wait` keeps a died run's last capture (D5).
- seatengine: each table declares one group, named by role prefix and round and sized by its seat count, and a one-seat table gets a window to itself (D11).
- `lyx shuttle open` carries the operator audience, so the command index lists it (D12 and the scope line on the index).
  Its provider specifics sit in `claudeengine` and its chain in `internal/shell`.
- `lyx reed add --anchor` accepts `own-window`.

Nothing outside the scope changed: no cross-worktree columns, no new key bindings, no zoom change, and no readiness-probe fix, which stays with `run-window-bugs`.
The open risk on detached-window size remains with `run-window-bugs`, as the record says.
The Plan-Review carry-over `card4-relaunch-kill-unlicensed-by-d6-stale-listing` is addressed by `9e5925edb`.

Card 13 states one deviation: `lyx shuttle open` uses the shuttle group's existing wiring rather than `internal/cliwire`, because no shuttle verb goes through `cliwire` today.
This is a wiring detail and leaves every recorded decision intact.

The `docs/overview.md` change also removes a duplicate shuttle bullet that was fused onto the original on `main`; card 13 names this cleanup.

## Overlap

`lyx loom peers` lists no landed parent commits and two runs in flight, `judge-cap-call` and `go-driver-assistant`.
Neither changes a Go package or code file this run changes, and this run changes no PATTERN entry.
The shared files are bookkeeping only:

- `cmd/lyx/testdata/test-budget.yaml` (both peers): this run edits the `reedengine`, `shell`, `shuttleengine` and `claudeengine` rows.
- `docs/overview.md` (`judge-cap-call`): this run edits the reed and shuttle module-table bullets and the CLI line listing `lyx shuttle` verbs.

## Superseded

None.
`run-window-bugs` depends on this run, and stays open by its own entry.
