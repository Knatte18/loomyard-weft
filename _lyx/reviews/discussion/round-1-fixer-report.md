# Fixer report — discussion, round 1

Every finding's premise was checked against the code (`spawn.go`, `hubworkspace_write.go`, `junctionnames.go`, `walk.go`, `codeworktrees.go`, `reconcile.go`, `add.go`, `checkout.go`, `portals.go`) and holds.
All changes are in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Changes

- F1, fixed as proposed: D3 now states that an unresolvable prime keeps degrading the spawn to the bare folder with its Warn, and only failures past prime resolution are returned as the spawn's error. The failing-regeneration criterion's prime-spawn clause was reworded to match.
- F2, fixed as proposed: D3 says a failure before the write writes nothing and a failed write may leave the file truncated or partial. D4's Bound adds the truncated file, which the next regeneration rewrites. The criterion drops failed write from its writes-nothing list, and a new criterion says a regeneration after a truncated file rewrites it whole.
- F3, fixed as proposed: D5 names `<hub>/_board/.lyx/hub-workspace.lock`, a 5-second wait, and no lock, no directory and no Bolt on a hub with no `_board`. The clone criterion adds that `_board` stays absent.
- F4, fixed as proposed: the D7 Warn trigger is keyed on fabric-shaped portal links, by the sweep's target-path rule, and on the retired workspace path. The Warn criterion adds the foreign-entry-only hub. The self-check narrowed the trigger to a file or link at that path.
- F5, fixed as proposed: D4 says the regeneration failure is not carried in the verb's result. D7 places hub-level details in a new `HubDetails []string` field on `ReconcileResult`.
- F6, fixed as proposed: D2 says the regeneration's listing skips an unresolvable entry at Debug, and `CodeWorktrees` keeps returning that failure for its other callers.
- F7, fixed as proposed: D4's Bound names a persistent cause, which keeps the list stale and the Warn repeating until the operator fixes it. D4's verb sentence has the Warn name the failure beside `lyx fabric reconcile`.
- F8, fixed as proposed: D7 states that Reconcile sweeps before it regenerates.
- F9, fixed as proposed: D6 retires the portal create, remove and restore helpers, and the sweep keeps the target shape and ownership check, moved beside it.
- F10, fixed as proposed: D7's second bullet keeps a directory at the retired workspace path with a detail and never removes it. The Bound now reads "the one file or link at the retired workspace path, never a directory", and the planted-symlink criterion adds the directory case.
- F11, fixed as proposed, taking the first option: `Checkout` joins D4's regenerating verbs on every exit past its pre-flight checks, rollback included. Scope's In list and a new criterion cover the branch switch.
- F12, fixed as proposed: the criterion reads "An unchanged file, folder list and settings both, rewrites nothing".

## Self-check

Six merged fork findings, all fixed:

- fix-consistency MEDIUM, fix-way-forward MEDIUM and fix-proposal MEDIUM (merged): the Warn trigger "an entry at the retired workspace path" fired forever on a kept directory. Fixed: the trigger is now "a file or link at the retired workspace path", and the Warn criterion adds that a hub whose retired path holds only a directory logs no Warn.
- fix-consistency MEDIUM, fix-proposal MEDIUM and fix-claims LOW (merged): the no-`_board` path contradicted the unconditional concurrency criterion, and the creation of `_board/.lyx` was unstated. Fixed: D5 creates `_board/.lyx` under an existing `_board` and says regenerations on a no-`_board` hub run unserialized. The concurrency criterion is scoped to a hub with a `_board`, and an Open risk names the gap.
- fix-consistency LOW: D4 stated the Warn's content twice. Fixed: the Bound's repeat clause is dropped.
- fix-consistency LOW and fix-proposal LOW (merged): D4's Rationale still said the first folder never changes, which the `Checkout` rename and a truncated file falsify. Fixed: the Rationale names both cases.
- fix-claims LOW: `runReconcile` was ambiguous between `fabriccli` and `configcli`. Fixed: it now names `fabriccli`'s `runReconcile`.
- fix-way-forward LOW and fix-proposal LOW (merged): the kept directory's detail had no stated home. Fixed: it is a hub-level detail, and D7 says every hub-level detail of the sweep lands in `HubDetails`.

## Disputed

None.

## Deferred

None.
