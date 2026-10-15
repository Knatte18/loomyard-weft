# Fixer report — discussion round 1

## Changes

- F1, fixed as proposed (the accept branch): D2's Bound now names every entry kind that re-arms follow (a fresh run, a re-entrant start on a live run, a resume of a parked run by the orch, batten's Run-Shed or an approve or reject follow-up), accepts the re-arm on resume, and states the bound as the entry itself.
  Added the acceptance row for a parked run resumed by `lyx loom start` with follow off.
  Premise checked: `internal/loomcli/approve.go:103` and `reject.go:139` name `lyx loom start` as the `resume` step.
- F2, fixed as proposed: added the acceptance row pinning D4 — with follow on, an overview present and a first client on the shell, a second attach moves the first client to the overview; with follow off it leaves it on the shell.
- F3, fixed as proposed: the overview add/replace/resume row now reads "changes no follow state and moves no viewer except D6's move off a window that ends".
  Premise checked: `internal/reedengine/replace.go:94` selects the replacement overview's window.
- F4, fixed as proposed: D4's last Rationale sentence is replaced by a Bound line saying with follow on an attach moves every attached client, including one moved outside reed's bindings (D5's bound), and with follow off moves none while another client is attached.
- F5, fixed as proposed (the second option): D3 now bounds the `Focus` override by an explicit focus flag and names `lyx shuttle run --focus` beside `lyx orch start` and `lyx reed add --focus`, dropping the "neither a run in a task pair" claim.
  Premise checked: `internal/shuttlecli/run.go:134` sets `Focus: focus`.
- F6, fixed as proposed: D7 decides the degraded click — when the D8 verb's command cannot be composed, a click from off only sets `@lyx_follow` on and selects no window, logged at Warn as for the session-switch keys.
  Premise checked: `switchBindingCommandsLocked` in `internal/reedengine/bindings.go` returns empty lines on an unresolved socket path, tmux path or binary.
- `support-log.md`: recorded round 1 under Review rounds.

## Disputed

None.

## Deferred

None.
