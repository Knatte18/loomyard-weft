# Card 6 — lyx fabric merge-in warns while a Webster run is in flight

**Edit:**
- `internal/fabriccli#addMergeVerbs`
- `internal/fabriccli#setMergeExit`
- `internal/fabriccli#errConflictsWithRecord`
- `internal/fabriccli#addWeftVerbs`
- `plugins/ly/skills/ly-drive/SKILL.md`

**Create:**
- `internal/fabriccli/mergein_webster_integration_test.go`

**Uses:**
- `plan:internal/websterengine#RunInFlight`
- `internal/lyxcwd#Location.AnchorPath`
- `internal/logger#Warn`

**Intent:**
Make the operator path that causes the wedge visible without blocking it (D6, D7).
The merge still proceeds in every case; this card never touches `fabricengine.MergeIn`, so landing's own parent merge-in through the engine is unaffected.

`addWeftVerbs` hands `addMergeVerbs` a second getter closure returning the `*lyxcwd.Location` its `PersistentPreRunE` resolved, for the same reason the fabric handle is a getter: the location is assigned after the commands are registered.
The `merge-in` RunE calls `websterengine.RunInFlight(loc().AnchorPath())` — the value `hubgeom.WebsterGeometry` hands webster as `AnchorRoot` — after `MergeIn` returns.
When it is true, the verb logs one `logger.Warn` and passes one warning to the envelope: Webster is mid-run in this worktree, `lyx webster record-batch` tolerates the parent merge, and integration bisect over earlier card SHAs will run on pre-merge trees.
The warning goes under a `warnings` string array on the ok envelope of a merge that moved HEAD (not `already_up_to_date`) and on the conflict envelope; a hard failure and an already-up-to-date result carry no `warnings` key.
A `RunInFlight` error degrades to no warning plus a `logger.Warn` naming the error, never a failure.
`setMergeExit` gains a warnings parameter that `merge --continue`, `merge --abort` and `merge <branch>` pass as nil, and `errConflictsWithRecord` gains the matching optional field, so their envelopes are unchanged.
`merge-in`'s `Long` help gains a paragraph documenting the warning and when it appears; `Short` is unchanged.

In the `ly-drive` skill's `## Repairing` section, add one short rule: prefer not to merge the parent into a task worktree while its Webster phase is in flight;
when a parent fix is needed mid-run, `lyx fabric merge-in` is survivable and will warn.
The skill must stay recipe-blind (`cmd/lyx/lydriveskill_test.go`): the rule may not contain the whole word "loom" or the literals `lyx loom`, `lyx batten` or `.lyx/`.

Tests, in the new integration-tagged file in package `fabriccli_test`, on a `hubforge.NewHub` pair driven through `fabriccli.RunCLIIn` with the existing helpers in `merge_cli_integration_test.go`.
Seed webster state by writing `state.json` (and `outcome.yaml` where the case needs one) under `websterengine.Dir(h.PrimeWorktree())` and committing it on the prime weft before the weft feature branch is cut, since `MergeIn` refuses a dirty pair:
- in-flight state (no outcome), clean diverging merge-in: exit 0, the merge commits, and `warnings` is exactly the one warning;
- in-flight state, conflicting merge-in: exit 1, the conflict envelope carries `conflicts` and the same `warnings`;
- no webster state: exit 0 with no `warnings` key;
- outcome `done`: exit 0 with no `warnings` key;
- in-flight state with an already-up-to-date merge-in: no `warnings` key;
- malformed `outcome.yaml`: the merge completes (exit 0, `committed` true) with no `warnings` key.
The existing merge CLI tests pass unchanged.

**ImpactSummary:** only `merge-in`'s envelopes gain an optional `warnings` key; the other merge verbs' envelopes and every exit code are unchanged; one rule added to the ly-drive skill.

**Commit:** 6: fabriccli: merge-in warns while a Webster run is in flight
