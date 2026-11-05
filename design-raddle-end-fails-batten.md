# A finished raddle run fails its batten run at Run-Shed

`raddle-end-fails-batten` · Note · bug, batten

## Seen

Raddle on creel-cross-hub-mail filed its entry and ended at Raddle-File, which marks the Sketch entry done. Batten then failed at Run-Shed with "origin ended ... (it landed and its board entry is done); run lym cleanup --apply" and the shed driver asked the orch to run it, instead of going on through Land and Worktree-Teardown. `lym batten status` gave a contradicting hint (`child_done_batten_halted`: run `lym batten run`).

## Workaround

`lym cleanup --apply` from the prime, then `lym batten cancel <slug>`.

## Expected

A raddle run that filed its parts goes through Land and Worktree-Teardown like any finished run, with no orch turn.