# A wake from hibernate checks what moved on origin and lets runs go on only when nothing did and a switch allows it

`hibernate-wake-resume-switch` · Note · enhancement, hibernate, tender, fabric

## Seen (operator, 2026-11-02)

A wake from hibernate lifts the hub-wide pause the hibernate added, so every paused run goes on at once.
Task `hibernate-wake-leaves-pause` (`44ca77de`) made that lift reliable: the daemon lifts its own pause on every path, before the notice.
The operator often does not want runs to start again by themselves after a wake.

## Idea

- Whether the wake lifts the pause is a switch: a key in the hibernate config gives the default, and `lym hibernate request` takes a flag that overrides it for one request.
- With the lift off, the hub stays paused after the wake, the wake notice to the orch says so, and `lym tender resume` lifts it.
  This changes the lift-always rule `44ca77de` set, and the switch's off path must keep the pause entry live while the request still reads `done`; `lym hibernate cancel` already lifts a pause a finished request left behind.
- The wake's other steps, its notice included, run either way.
- The default is off (operator, 2026-11-02): the operator often works on another machine while this one sleeps, and when origin or a run's owner moved meanwhile, going on is wrong.
- The wake detects remote change by itself, whatever the switch says:
  - At the hibernate, lym records the tips of the prime's code and weft branches on origin and the owner of root, of each beam and of each run.
  - At the wake, it fetches both repositories and compares origin's tips and the owners with that record.
  - The wake notice lists what moved: the branches whose tip changed, with the commit count, and each owner that changed, with its new machine.
    A notice stays one short line since `44ca77de`: a longer list is written whole to `notice.txt` in the hibernate directory, and the queued line names that file.
- Even with the switch on, the wake keeps the pause when anything moved, and its notice says so and names `lym adopt` when an owner moved away.
