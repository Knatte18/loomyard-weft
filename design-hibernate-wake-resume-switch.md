# A switch decides whether a wake from hibernate lets the runs go on

`hibernate-wake-resume-switch` · Note · enhancement, hibernate, tender

## Seen (operator, 2026-11-02)

A wake from hibernate lifts the hub-wide pause the hibernate added, so every paused run goes on at once (task `hibernate-wake-leaves-pause` makes that lift reliable).
The operator often does not want runs to start again by themselves after a wake.

## Idea

- Whether the wake lifts the pause is a switch: a key in the hibernate config gives the default, and `lym hibernate request` takes a flag that overrides it for one request.
- With the lift off, the hub stays paused after the wake, the wake notice to the orch says so, and `lym tender resume` lifts it.
- The wake's other steps, its notice included, run either way.

## Open

- The default of the key.
