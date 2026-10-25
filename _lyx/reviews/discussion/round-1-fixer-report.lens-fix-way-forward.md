```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked that each refusal, next step and recovery the fix added can be carried out from the state it names and leads to progress.

Ways forward checked:
1. The D7 owner refusal and its exemption for taking up the upstream's anchor.
2. The D7 parent-lock and child-pass refusal: "wait … or `lyx tender pause <child>`".
3. A D7 re-run with a different `--set` while the record stands: refused, naming the record's `B`.
4. D7 recovery when both `A/.lyx` and `B/.lyx` exist: merge by hand.
5. The D7 kill after step 8: the next weft push and `lyx beam start`.
6. The D8 reconcile refusal on a standing move record: `lyx fabric anchor <slug> --set <B>`.
7. The D10 pull and push stop: `lyx fabric anchor <slug> --set <anchor>`.
8. A D10 integration conflict: "move the conflicting local changes out of the weft, re-run, and write them back at `T`".
9. The D7 move record that makes `EnsureLive` skip the worktree.
10. The Open risks operator step: reconcile first after updating.
11. An integrating `--set` on a second machine passing D7's validation of `B`.

Findings:

- **MEDIUM** (D10, integration paragraph, conflict way forward): the way forward makes no progress.
  The conflicting changes are local *commits* that `U` lacks, and the weft is left unchanged.
  Moving the changes out of the weft takes a further commit on top of them.
  The re-run's replay still replays the original conflicting commit before reaching that one, so it stops at the same place.
  Evidence: "It replays each local commit `U` lacks … a conflict stops it with the weft unchanged".
  For the re-run to get past the conflict, the way forward must drop or rewrite the conflicting commit out of the local branch, and it names no way to do that.

- **MEDIUM** (D8's move-record refusal, and D7's "Recovery from a kill is a re-run"): a standing record doesn't tell a live run from a killed one.
  "The verb holds no lock" against anything, so a reconcile run while a change is still running reports the pair failed and names `lyx fabric anchor <slug> --set <B>`.
  Following that way forward starts a second run alongside the live one.
  The re-run reads the same record and runs steps 2 to 8 concurrently with the live run.
  Evidence: D7: "A standing move record is the evidence that a change is in flight"; nothing names a lock or pid that a re-run checks.

- **MEDIUM** (D7 step 1: "While it stands, `dobbyengine.EnsureLive` starts no dobby of the worktree, and none at all for the prime"): no way forward is named for the skip.
  After a killed run, the record stands until someone re-runs the verb.
  Until then `lyx orch start`, `lyx adopt` and `lyx beam start` start no dobby, for the prime on the whole machine, and nothing says what the operator is told.
  `PATTERN-refusal-way-forward` asks that the skip entry name `lyx fabric anchor <slug> --set <B>`.
  The same gap applies to D7's "the dobby start, which `lyx beam start` makes": that holds only after the record is gone, which that sentence does assume.

- **LOW** (D7 refusal: "or `lyx tender pause <child>` first so no new full pass of it starts"): the child is left paused.
  `lyx tender pause [<slug>]` also ends the run's processes (internal/tendercli/doc.go:31), so the pass does stop and the re-run can proceed.
  But no step or way forward names `lyx tender resume <child>` after the change.
  The claim "so no new full pass of it starts" also understates what the command does: it ends the pass in flight.

- **LOW** (the integrating `--set` on a second machine, and the new acceptance criterion "its `lyx fabric anchor <prime> --set B` integrates"): D7's validation can refuse it.
  Validation requires `B` to be "a directory git tracks in the worktree's warp checkout".
  On a machine whose warp checkout predates the commit that added `B`, the integrating run is refused by validation.
  Neither D7 nor D10 names pulling the warp first as the way forward.

Tool calls:
- Ran `grep` over `internal/tendercli` for the `lyx tender pause` usage and its end behaviour, which my context did not show.
