```yaml
lens: code-affected-set
outcome: reported
```

**Lens: code-affected-set.** For each change, here is what I found that calls, reads or asserts it, and whether the diff covers it.

1. **`finalizeEnteredRead` now also counts a pair status at `Raddle-File`.**
   - Callers: `withoutFinalizeEntered` has three call sites. They are `replay.go:407` (replay), `replay.go:661` and `originreconcilemoves.go:597` (the origin reconcile moves).
   - The loom pre-push check and the batten resume read the ended set from that reconcile, so they inherit the exemption.
   - Tests: the `originreconcileended` and `replay_integration` rows cover it.
   - The `fabricengine/doc.go:1302` paragraph landed.
   - **Not covered:** two prose descriptions of the ended exemption are left stale (findings 2 and 3).

2. **`ClearMotifRun` has a second caller (the raddle start), and its commit message changed from `beam create:` to `batten:`.**
   - No test or doc asserts the old message. The grep matches only `beamcli/create.go:187` (a different error string), `cleanup.go:158` and `startguard.go:441`.
   - The `cleanup.go` file header, godoc and help text, the `cli.go` doc and the `startguard.go` header were updated. Covered.

3. **Dobby's `StartRefusal` is now skipped for a done motif run of a raddle task.**
   - Consumers: `dobbyengine/view.go:68` and `starts.go:63`, `starts.go:115` and `starts.go:352`, and `battencli/overviewdobby.go:139`. All read the field only, so they need no change.
   - **Not covered:** the prose describing the field is stale (finding 1).

4. **`sketchFinish` gained a `notify` parameter.**
   - The only callers are `wiring.go:222` and the new test at `commitstatus_integration_test.go:145`. Covered.
   - `loomcli/doc.go` was updated.
   - The notice's `lym board set-deps`, `upsert` and `set-status` verbs exist in `boardcli` (`cli.go:4`).
   - The notice names no denied form (`PATTERN-refusal-way-forward`).
   - I found no stencil or spec outside `doc.go` that describes the refused-Sketch path. The grep matched only `raddle-proposal-spec.md`, `docs/overview.md` and `raddle-recipe.yaml`, and only on `Raddle-File`, which I did not open.

5. **`PATTERN.md` and `pattern/`.**
   - My context shows no declared PATTERN changes, so there is nothing to check as landed.
   - The commits carry no `Pattern-Departure:` trailers and none is needed.

**Findings**

1. LOW, `consistency`. `internal/dobbyengine/doc.go:109` says a released task whose slug has a finished run record carries the refusal every start gets. `DobbyTasks`' godoc at `internal/battencli/dobbyreads.go:287` says the same. Both are now wrong for a raddle task whose done run's child ran motif, which `dobbyreads.go:325-333` exempts. The diff changed the behavior but not these two statements. Evidence: `dobbyengine/doc.go:109`, `dobbyreads.go:287` and `dobbyreads.go:325-333`.

2. LOW, `consistency`. The `guard` godoc at `internal/loomcli/prepush.go:113` says an entry origin ended pauses the run "unless its parent already holds the run's work". That is no longer the whole exemption. `finalizeEntered` now also exempts a raddle run standing at `Raddle-File`, which never has parent work. Evidence: `prepush.go:113` and `replay.go:250`.

3. LOW, `consistency`. `internal/loomcli/doc.go:290` has the same stale sentence: "a run origin ended is paused naming `lym cleanup --apply`, unless its parent already holds its work". I saw only the grep line, not the surrounding paragraph. The diff updated `doc.go:25-28` in that package but not this sentence. Evidence: `loomcli/doc.go:290` and `replay.go:227`.

4. NIT, `consistency`. `internal/fabricengine/replay.go:38` adds `raddleFileRow = "Raddle-File"`, which repeats the literal behind `loomshed.NameRaddleFile` (`loomshed.go:52`). It follows the existing `finalizeRow`, `landRow` and `worktreeTeardownRow` constants in the same block. Fabric cannot import `loomshed` without breaking `PATTERN-fabric-boundary`'s isolation rule, so the repeat is probably accepted debt. It adds one more loom row name to fabric's knowledge, which that entry says never to extend. Evidence: `replay.go:35-40` and `PATTERN-fabric-boundary.md` ("Fabric isolation").

Tool calls (5; I used 3 greps for things my context lacked and 1 read):
1. grep for the old commit message string: to find any test or doc that asserts it.
2. grep for callers of `sketchFinish`, `withoutFinalizeEntered` and `finalizeEntered`: my context had only the changed definitions, not all callers.
3. grep over `.md`, `.yaml` and `.tmpl` files for docs or specs describing the changed behavior: to find stencil or spec text.
4. grep of `dobbyengine`, `battenshed`, `battencli` and `contracts` docs for done-run refusal statements, plus the non-test uses of `finalizeEntered` in `fabricengine`: to find stale statements.
5. read of `prepush.go:108-138`: to see how the pre-push check consumes the ended verdict.
