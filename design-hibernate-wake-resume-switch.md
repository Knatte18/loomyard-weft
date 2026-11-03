# A wake from a lym hibernate lifts the pause only when a switch allows it

`hibernate-wake-resume-switch` · Task · enhancement, hibernate, tender · draft

## Seen (operator, 2026-11-02)

A wake from a hibernate that `lym hibernate request` put the machine into lifts the hub-wide pause that hibernate added, so every paused run goes on at once.
Task `hibernate-wake-leaves-pause` (`44ca77de`) made that lift reliable: the daemon lifts its own pause on every path, before the notice.
The operator often does not want runs to start again by themselves after a wake.

## Scope (operator, 2026-11-03)

This entry covers only a hibernate lym started through `lym hibernate request`.
A sleep the machine starts by itself adds no pause and needs none: when nothing moved meanwhile, runs go on as they were.
Work that moved on another machine meanwhile is caught where it matters, when a run commits, pushes or lands; that check is a note of its own, not part of the wake.

## Idea

- Whether the wake lifts the pause is a switch: a key in the hibernate config gives the default, and `lym hibernate request` takes a flag that overrides it for one request.
- The default is off (operator, 2026-11-02): the operator often works on another machine while this one sleeps.
- With the lift off, the hub stays paused after the wake, the wake notice to the orch says so, and `lym tender resume` lifts it.
  This changes the lift-always rule `44ca77de` set, and the switch's off path must keep the pause entry live while the request still reads `done`; `lym hibernate cancel` already lifts a pause a finished request left behind.
- The wake's other steps, its notice included, run either way.
