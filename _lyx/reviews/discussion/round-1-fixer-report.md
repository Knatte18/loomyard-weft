# Fixer report — round 1

All edits are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

- F1: the `/compact` survival premise is now an Auto-mode assumption and an Open risk, D7 and D11 mark it unverified, and the compact smoke criterion asserts a background completion arrives.
- F2: D13 caps the queue at 50 (oldest dropped, logged), drops it at watcher start when no strand is recorded, and states the dead-strand bound; D12 persists episode state so a batten restart does not re-notify.
- F3: D12 defines the `[batten notice]` prefix, one-line rendering, control-character stripping and a 200-character cut; D14 uses the literal; criteria added.
- F4: D10 names strict `configengine.Load`, the surfacing points (orch start, watcher start, before any pane) and that a running watcher is unaffected; criterion added.
- F5: D2 states the bound for a requested cycle.
- F6: D15 names the refusal message when the start is unreadable; Open risk added for an absent registry start.
- F7: D16 states the stuck reason clears on the next passing probe; criterion updated.
- F8: the D12 quiet condition is restricted to a running child; criteria added for `notice_quiet_min` and the one-line compact stencil.
- F9: the process-narrative assumption is removed, and the board amendment moved from Open risks to Constraints as a landing obligation.

## Deferred

None.
