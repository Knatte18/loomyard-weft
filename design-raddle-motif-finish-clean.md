# Raddle and motif runs start and finish without the orch

`raddle-motif-finish-clean` · Task · bug, batten, dobby, loom, board, orch · running · Code-Burler-Review

## From note raddle-same-slug-completed

### Seen (orch, 2026-11-05, twice)

Promoting a Sketch gives the same entry recipe raddle under the same slug.
For `reed-slack-relay` and again for `creel-cross-hub-mail`, dobby could not start the raddle run: battencli reported the slug `has already completed` and named `lym batten cleanup --apply`, since the motif run that made the Sketch left its run directory under `_lym/shed/<slug>`.
The orch ran the cleanup each time and released the task again.

### Idea

The promotion of a Sketch to raddle, or batten at Seed of a raddle run, clears the finished motif run's directory of that slug, as the cleanup does, so the raddle run starts with no hand step.


## From note raddle-end-fails-batten

### Seen

Raddle on creel-cross-hub-mail filed its entry and ended at Raddle-File, which marks the Sketch entry done. Batten then failed at Run-Shed with "origin ended ... (it landed and its board entry is done); run lym cleanup --apply" and the shed driver asked the orch to run it, instead of going on through Land and Worktree-Teardown. `lym batten status` gave a contradicting hint (`child_done_batten_halted`: run `lym batten run`).

### Workaround

`lym cleanup --apply` from the prime, then `lym batten cancel <slug>`.

### Expected

A raddle run that filed its parts goes through Land and Worktree-Teardown like any finished run, with no orch turn.

## From note motif-sketch-refusal-silent

### Seen (orch, 2026-11-05)

The motif run `creel-cross-hub-mail` ended at 16:33 UTC with its decision record stored and its pair torn down.
`sketchFinish` in `internal/loomcli/wiring.go` could not make the entry a Sketch, since `slack-relay` depends on it, so it left the entry a draft task with `discussed: true` and logged a Warn.
No mail reached the orch; the orch learned of it from dobby's board tidy report an hour later.

### Idea

When the board refuses the Sketch, mail the run's parent naming the refusal, the entries that depend on the entry, and the way forward: drop or move those dependencies and make it a Sketch, or set recipe raddle on the discussed task.


