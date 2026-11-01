# A hub rename program must tolerate its own landing run and the new binary

`boltrename-landing-order` · Note · bug, cwd, fabric

## Seen (orch, 2026-11-01)

`bolt-dir-rename` landed a lym binary that knows only `_bolt` while the hub was still on `_board`, and its one-shot `tools/boltrename` could not run as shipped:
- lym processes on the new binary (dobby, the reed watchdog) made an empty `_bolt/.lym` before the migration, and the program refuses a hub holding `_bolt`;
- the landed run's own pair was still in the hub, since teardown comes after Land, and the program refuses any directory but the prime, its weft, `_launchers` and the Bolt checkout.
Neve was migrated by hand around both; the program is being deleted.

## Lesson for a later rename

A program that renames a hub's directories after a rename task lands must tolerate the landed run's own pair and state the new binary has already written, or the task must say to run it before anything starts the new binary and before the run's teardown.
