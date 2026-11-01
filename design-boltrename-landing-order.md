# boltrename refused the hub its own landing left: a stray _bolt and the run pair

`boltrename-landing-order` · Note · bug, cwd, fabric

## Seen (orch, 2026-11-01)

`bolt-dir-rename` (commit `229638f7`) landed a lym binary that knows only `_bolt` while the hub was still on `_board`, and its run then stuck at Worktree-Teardown (board config under `_bolt` not found).
`tools/boltrename` could not run as shipped:
- lym processes started on the new binary (dobby, the reed watchdog, a reed discover signal) made an empty `_bolt/.lym` before the migration, twice; boltrename refuses a hub holding `_bolt`.
- The landed run's own pair (`bolt-dir-rename` and its weft) was still in the hub, since teardown comes after Land; boltrename refuses any directory but the prime, its weft, `_launchers` and the Bolt checkout.
- Every live process in the hub blocks it, including an idle tmux server reed made and a plain shell.

Done by hand on Neve: the stray `_bolt` removed, the pair moved out of the hub, boltrename run, the pair moved back, `lym batten run bolt-dir-rename` finished the teardown.

## For OSL-1046

The same steps apply there, unless no run pair is in its hub.
A rename task's landing order (deploy at Land, teardown after) is worth stating in the rename program's own instructions, or the program should tolerate a landed run's pair and an empty `_bolt/.lym`.
`boltrename` is deleted once OSL-1046 is migrated, so this may only be a note for the next such rename.
