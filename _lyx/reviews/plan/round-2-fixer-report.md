# Fixer report — plan, round 2

## Changes

- F1 (MEDIUM, decision), fixed as proposed with option (a).
  The premise holds: D5 and D6 in `_lyx/discussion/decision-record.md` allow an own-window pane kill only on its strand's removal, and card 3's `planReconcile` ends an `@lyx_own` window once none of its panes is bound.
  Option (b) needs an amended decision record, which is outside this round's write surface and an operator decision, so (a) is the one sound change.
  - `_lyx/plan/04-reed-own-window-launch.md`: the `launchStrandLocked` own-window bullet drops the kill of the relaunching strand's recorded dead pane, its select-window step, the deliberate-extension sentence and the never-killed-or-ended claim.
    It now says a relaunch kills nothing, citing D6, and that the old dead pane is left unbound and its window is ended by card 3's reconcile, after D5's client select, once none of its panes is bound.
    `selectStrandsWindowIfCurrentLocked` leaves the card's `**Uses:**`, since nothing else on the card calls it.
    The `doc.go` "Own windows" sentence now says the dead pane stays until the strand is removed or, once a relaunch leaves it unbound, until its window holds no bound pane.
  - `_lyx/plan/05-reed-window-groups.md`: the own-window branch heading and the re-list sentence no longer order anything after card 4's kill.
    `RelaunchOfADeadGroupMemberRejoinsItsWindow` now asserts the resulting arrangement: the relaunch joins the group window beside the live partner and the old dead pane stays unbound and unkilled; with the partner removed first, the group window stays because the dead pane is still bound until the relaunch, the relaunch joins it, and a later `RemoveStrand` of the relaunched strand ends the window.

## Disputed

None.

## Deferred

None.
