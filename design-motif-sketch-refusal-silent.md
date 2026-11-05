# A finished motif whose Sketch the board refuses stays a draft task with no notice to the orch

`motif-sketch-refusal-silent` · Note · bug, loom, board, orch

## Seen (orch, 2026-11-05)

The motif run `creel-cross-hub-mail` ended at 16:33 UTC with its decision record stored and its pair torn down.
`sketchFinish` in `internal/loomcli/wiring.go` could not make the entry a Sketch, since `slack-relay` depends on it, so it left the entry a draft task with `discussed: true` and logged a Warn.
No mail reached the orch; the orch learned of it from dobby's board tidy report an hour later.

## Idea

When the board refuses the Sketch, mail the run's parent naming the refusal, the entries that depend on the entry, and the way forward: drop or move those dependencies and make it a Sketch, or set recipe raddle on the discussed task.
