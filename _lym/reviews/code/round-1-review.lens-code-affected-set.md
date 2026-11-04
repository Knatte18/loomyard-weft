```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. Per change, I list what reads or asserts it and whether the diff covers it.

## Per-change list

1. **Creel accepts a post to an unlisted hub orch (`isHubOrchName`, `checkReceiverLocked`).**
   - Set found: the removed-holder sweep in `sweep.go`, `Store.sweepSubjects` and `releaseGoneHolder`. The creel `doc.go` sweep section and the dobby `doc.go` claim "waits … delivered once a strand of that name is listed". The new `verbs_test.go` row.
   - **Not covered.** See F1.
2. **`Store.Prune` keeps a done entry carrying `notify_on_land`.**
   - Set found: three prune callers: `boardcli` `prune`, `battencli/land.go` `PruneBoard` and `cleanupcli/segments.go` "board" segment. Their help text and docs: the `lym board prune` `Long` text, `docs/overview.md`, `contracts/orchrole/orch-role.md`, and `plugins/ly/skills/board/SKILL.md`.
   - **Partly covered.** The `boardengine` doc and the `Prune` doc comments are updated. The user-facing text is not. See F2.
3. **`notify_on_land` can now be set on a done entry (upsert notice, `landedNotice`).**
   - Set found: the `lym board upsert` help text, updated in the diff, and the `boardengine/doc.go` paragraph, also updated. `plugins/ly/skills/board/SKILL.md` has no mention of the field.
   - Covered.
4. **`TaskView.Parent` is now read for a done entry carrying the field (`DobbyTasks`).**
   - Set found: the `TaskView.Parent` comment, updated. The consumers `released()` in `starts.go:58` and `landnotice.go:104`. `released()` also checks an empty status, so it is unaffected.
   - Covered. See F3 for the parent fallback.
5. **`LandNoticeState` loses `Attempts` and gains `Line` and `ClearOwed`.**
   - Set found: `view.go:106`, which reads only `Requested`. The state record is covered by `PATTERN-tolerant-records`: an old record's `attempts` key is kept by `internal/state`. The `dobbyengine/doc.go` landing-notice section was updated. The run-notice line in the same `doc.go` (line 142, "at most three times") is a separate mechanism and still true.
   - Covered.
6. **New `boardengine` exports `StatusDone` and `IsDoneStatus`.**
   - Set found: `isDone` was rewritten onto them. The `"done"` literals elsewhere were not touched.
   - Not a finding. The other literals are the diff's choice not to migrate, and `PATTERN-named-constants` forbids repeated literals only for a name that can go stale.
7. **New `dobbyengine` import of `boardengine` in `landnotice.go`.**
   - Set found: `starts.go` already imports `boardengine`, so the import edge is not new.
   - Covered.
8. **PATTERN and commit rules.**
   - No `Pattern-Departure:` trailer is present, and the task's directive declares no PATTERN changes, so none is owed.
   - `PATTERN-parent-lock` and `PATTERN-front-task` are not touched.
   - No stencil changed.

## Findings

**F1: BLOCKING, design. Mail to an absent orch is returned undelivered by the creel sweep, so it does not wait for the orch.**
- Location: `internal/creelengine/pool.go` (`checkReceiverLocked`), against `internal/creelengine/sweep.go`.
- A post to an unlisted `<shortname>:orch` is stored with that name as its holder.
  - `sweepSubjects` includes the holder, because `isStrandName` is true for an orch name.
  - `sweepOnce` finds it in no strand listing and counts a miss.
  - After `goneAfterSweeps` (2) misses, about 30 s plus the 10 s confirm, `releaseGoneHolder` runs.
  - The unread message goes back to its sender, which is a program (dobby).
  - `isStrandName(next)` is false for a program, so the message is journaled `undelivered`.
- That is the opposite of "stays unread in the pool and is delivered once the orch's strand is back" in the creel `doc.go`, the dobby `doc.go` and the summary. It also defeats the goal's item 3 for any orch absent longer than about 40 s.
- Dobby sees the post succeed, clears the request and drops the owed line, so the notice is lost silently.
- The only test (`verbs_test.go`, the new `Post_ToAnOrchOfTheHub…` row) lists the strand before any sweep runs. No test combines an unlisted holder with the sweep (`deliver_test.go` `TestSweep_RemovedHolders`).
- Evidence: `sweep.go:134-148`, `sweep.go:167-171`, `sweep.go:179-186`, `sweep.go:206-230`.
- Fix direction: exempt a hub-orch holder from the gone-holder sweep while its strand is unlisted, and pin it with a sweep test.

**F2: LOW, consistency. Prune's user-facing wording still says it removes every done entry.**
- Location: `internal/boardcli/cli.go:256-259`, the `prune` `Long` text ("removes every task whose status is done"). Also the flag help at `cli.go:275`.
- Related prose that the diff left stale: `docs/overview.md:721` ("prunes the done entries off the board"), `contracts/orchrole/orch-role.md:193` ("prunes the board") and `plugins/ly/skills/board/SKILL.md:185` ("`lym board prune` removes it").
- A flagged done entry now survives until dobby clears the field.
- The `Long` text is the one an operator reads when a landed entry stays on the board after `prune --apply`. The rubric's "docs the diff's kind of change names land in the same diff" applies.
- Evidence: the diff changes only the `Prune` doc comments and `boardengine/doc.go`.

**F3: LOW, design. A done entry whose origin record is gone falls to the prime's dobby.**
- Location: `internal/battencli/dobbyreads.go:287-296`, with `internal/hubgeom/parentof.go:26-31`.
- `DobbyTasks` now calls `parents.Of` for every entry carrying `notify_on_land`. With no origin record, `ParentOf` answers through `ParentBeam` on the board. When the beam is no longer open it returns `""`, and `DobbyTasks` maps `""` to the prime.
- So a beam member that landed, whose pair and origin record were torn down, may be noticed by the prime's dobby instead of its beam's. That sends the line to the prime orch rather than the beam's orch.
- `Of` returns no error for an absent record, so the read does not abort `DobbyTasks`.
- No test pins this fallback. I did not trace whether Land's teardown actually removes the origin record before the dobby's next tick.

## Tool calls
1. `grep` over `docs`, `contracts` and `_bolt` specs, `internal/dobbyengine` imports, `.Parent` consumers and the prune help text. My context lacked the documentation and consumer set.
2. `grep` for prune and notice wording in `plugins/ly/skills`, `contracts/orchrole` and `docs/overview.md`. I needed to see whether those user-facing texts describe prune.
