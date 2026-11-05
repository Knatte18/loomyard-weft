```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the F1, F3, F4, F6, F7 and F8 searches. The fix covers every code site. I found two stale doc sets it did not reach, both LOW.

**Unchecked findings:** F2 and F5. Their premises are single missing tests, not sets, so there is nothing to search.

**Findings**

1. **LOW** (F7 sibling docs): `internal/battenshed/doc.go:60` ("An entry origin ended refuses, naming the landing or the removal and `lym cleanup --apply`") and `internal/landingshed/doc.go:150` ("A run origin ended is a Stuck…") state the ended-origin rule with no Raddle-File exemption.
   - The search for "origin ended / ended set" docs shows `loomcli/doc.go:290` and `fabricengine/doc.go:1302` updated, and these two siblings not.
   - Both read the report the reconcile builds, which no longer lists a done raddle run at Raddle-File. A reader of either doc would still expect a refusal for that run.

2. **LOW** (F1 sibling docs): the done-motif-run clear is documented in places that name only `lym beam create` as its caller.
   - `internal/battencli/cleanup.go:524` is the cleanup help text: "A done motif run is also cleared by `lym beam create`".
   - `docs/overview.md:713` says the same.
   - `cleanup.go:2`, `cli.go:12` and `cli.go:221` already mention the raddle start. These two omit it, though `ClearMotifRun`'s set of callers now includes the guard's `claimStart`.

**Covered, no finding**
- **F1:** `ClearMotifRun` has these callers: the guard seam in `hubStartGuard`, which only `claimStart` calls; `armAt` for both `Arm` and `ArmAt`; and `beamcli`'s own use. `refuseAtStart` now only reads. `ArmAt` goes through `armAt`, so it is covered. The `beamcli` clear (`create.go:185`) is a separate path outside the review's premise; I did not check whether it clears before an owner check.
- **F3:** `finalizeEntered` and `finalizeEnteredRead` are called only from `withoutFinalizeEntered`, which now passes `how`. `replayWeft` and `replayBolt` share that function, so both are covered.
- **F4:** `startClearsMotifRun` has two callers, `clearsMotifRunOfRaddleEntry` and `DobbyTasks`, and both now resolve the recipe through the board. The other `task.Recipe == RecipeRaddle` sites (`boardengine/sketch.go:111`, `labels.go:57`, `board.go:128`) are promote and label logic and unrelated to the start clear.
- **F6:** `DobbyTasks` godoc and `dobbyengine/doc.go:109` are updated. The `StartRefusal` field comments in `dobbyengine/view.go:50` and `starts.go:32` say "such as for a finished run record" and stay true.
- **F8:** `sketchRefusedNotice` has only the one site, and no other notice carries the bare "with `lym board upsert`" phrase.

**Further tool calls (3)**
- A grep of `ClearMotifRun` and related symbols in non-test Go, plus a grep for done-motif-run docs. My context lacked the full set of call sites and docs.
- A grep of `startClearsMotifRun` and recipe readers, plus a grep of `finalizeEntered` and `Raddle-File`. My context lacked the other recipe readers and the `fabricengine` and `prepush` sites.
- A grep of `StartRefusal` and `doneSlugRefusal` docs, a grep of ended-origin docs, and a grep of the upsert phrase. My context lacked the sibling docs for F6, F7 and F8.
