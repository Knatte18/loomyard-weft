---
verdict: APPROVED
---

## Scope check

The board entry (#633) names three bugs and their expected behaviour; the decision record covers each one.

- #699, a lost success envelope: D1 checks the success write on `runVerb`'s shared path, exits 1 and writes one `creel: ` line to stderr, for every verb that answers through `runVerb`, including the post, inject, log and inbox the entry names.
  D2 makes the post and inject line name the stored id, as the entry asks.
- #700, the doubled prefix: D3 prepends `creel: ` only when the message lacks it, giving one prefix as the creelcli `doc.go` line describes, and keeps `creel: creel read: ...`, which the entry calls intended.
- #701, `log --id` without reminders: D4 matches an event whose `Unread` holds the id, so posted, reminded and read come back in one call.

The change to `Store.Log` lies in `internal/creelengine`, not `internal/creelcli` as the entry's goal line says, but that is where the entry's own stated cause for #701 lives.
D2 also names the id in the fallback of non-zero post and inject results, a little past the entry's success-only wording.
It reuses the same field for the same stored state, so it adds no new direction and takes no other entry's work.
The Out list (the daemon verb, `internal/output`, the engine error texts, the id on a wait timeout) excludes nothing the entry asks for.

## Overlap

- `stencilstore-dev-warning-noise` (#626, darn, Darn-Write): both concern stderr on creel verbs, but that entry silences the stencilstore dev-build warning and this run changes creel's own refusal and lost-answer lines.
  The two runs change different behaviour, and the decision record names that warning as an open risk without changing it.
  The peer has no changed paths yet.
  Landing order is unknown: the peer's `row` is 0 and this run's `own_row` is 3.
- `pr-gate-run-identity` and `gate-wait-cache-cap`: none of their changed paths are in `internal/creelcli`, `internal/creelengine` or `tools/sandbox/`, and neither changes creel behaviour, so neither overlaps.
- Parent commits: `lyx loom peers` lists none.
- The later creel stages `creel-notices` (#552) and `creel-agents` (#553) move callers onto creel, and none of them plans a change to these output paths or to `Store.Log`.
