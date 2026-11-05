```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I walked each changed function with one concrete input and checked the added tests against the defect they guard. The gate was green at both tiers.

- **`refuseAtStart`**
  - A done status with `ClearsMotifRun` true takes the path `found=false`, a zero status, `motifRunClearPending=true`, then `guard.refuse` as a fresh start. A guard refusal returns before any removal.
  - A read error returns through `probeRefusal`, which wraps the error with `%w` and names the re-run. `TestRefuseAtStart_JudgesADoneMotifRunAsTheFreshStartItsClearMakes` fails if `refuseAtStart` calls `ClearMotifRun`, because it asserts `removed` stays false.
- **`claimStart`**
  - Pending with the status file present still runs the owner check; with `owner: "other"` it returns `OwnedElsewhereError` and leaves the pending flag set with no removal. With `owner: "this"` it removes once and clears the flag. A removal error wraps the cause, names the re-run and leaves the flag set.
  - `goto` returns early, which matches the old `run`/`step` gate. A `os.Stat` error other than not-exist skips the owner check, as before.
  - `TestClaimStart_RemovesTheDoneMotifRunOnlyOnceTheOwnerCheckPasses` fails if the owner check were skipped or the removal ran first. The row `ADoneRunTheStartDoesNotReplaceIsNeitherPulledNorRemoved` asserts zero pulls.
- **`startClearsMotifRun` and `clearsMotifRunOfRaddleEntry`**
  - A board that does not read returns an error and no removal.
  - An entry with no recipe under a raddle default now clears. `TestClearsMotifRunOfRaddleEntry` fails under the old `task.Recipe` read, and the dobby rows fail the same way. Both also assert the run is left in place.
- **`DobbyTasks`**
  - The caller keeps the `(location, task)` signature. A failed read warns and holds the start refusal, as before.
  - `OpenHub` now runs per done released task. Existing rows still pass because `writeBoardHub` writes the board config.
- **`finalizeEntered` and `finalizeEnteredRead`**
  - The only caller is `withoutFinalizeEntered`, which passes `how`.
  - With `how == "removed"` and a pair at Raddle-File, the pair block does not return true. The prime status `Implementer` also reads as not entered, so one teardown call results. The new reconcile row fails if the exemption ignored `how`.
- **`sketchFinish`**: the failed-notice row returns nil after one attempted notice. It fails if the notice error were returned.
- **`arm.go`**: the `io/fs` import has no other user, since vet and the gate compile.

Not findings, since the rubric excludes both:
- `sketchRefusedNotice` now carries `{"slug":S,"kind":"note","labels":["sketch"],"status":null,"recipe":"","depends_on":[]}`. This should pass the board's note rules: `validateWrite` refuses a note only for non-empty `depends_on` or for dependents on the entry, and `ApplyPatch` is a JSON merge. The test asserts the text only; nothing runs the payload against a board.
- The payload sets `labels` to just `["sketch"]`, so it drops the non-type labels that `MakeSketch` keeps. This is a LOW difference.

Calls made:
1. `grep validateWrite` in `internal/boardengine/store.go`, to confirm the F8 note payload passes note validation. My context held only `ApplyPatch` and `MakeSketch`, not `validateWrite`.
