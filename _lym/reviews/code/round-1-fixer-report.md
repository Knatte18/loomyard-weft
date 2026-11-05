# Fixer-report, code review round 1

Every finding F1–F8 was fixed; none is disputed or deferred.
`lym gate test` is green at tier 0 and tier 1 for `internal/battencli`, `internal/fabricengine`, `internal/loomcli` and `internal/battenshed` after the last change.
Fixes were committed per file group rather than one commit per finding, since F1, F2, F4 and F6 share `startguard.go`, `startguard_test.go` and `dobbyreads.go`, and F5 and F8 share one test file.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  - `refuseAtStart` no longer removes anything. It asks a new read-only guard seam, `ClearsMotifRun`, built over `startClearsMotifRun` and the board entry of the slug.
  - When the answer is yes it judges the start as fresh, so the raddle, unreleased, dirty-source and left-pair checks still run, and records the pending removal in `battenCLI.motifRunClearPending`.
  - A new `claimStart` method replaces the owner-check block in `armAt`. It runs the owner check when the status file is absent or a removal is pending, and removes the run through `ClearMotifRun` only after that check passes and before `armSeed`.
  - A failed removal returns the way-forward error naming the re-run.
  - The tests are `TestRefuseAtStart_JudgesADoneMotifRunAsTheFreshStartItsClearMakes`, which asserts `refuseAtStart` never calls `ClearMotifRun` even when the guard refuses, and `TestClaimStart_RemovesTheDoneMotifRunOnlyOnceTheOwnerCheckPasses`, which covers a refused owner check, a passing one, a failed removal and the no-pull rows.
- **F2 (MEDIUM), fixed with a change of the proposed assertions.**
  - `TestClearsMotifRunOfRaddleEntry` drives `clearsMotifRunOfRaddleEntry`, the renamed closure, over a fixture hub board and run directory.
  - Rows: a slug absent from the board, a board that does not read, a darn entry, a raddle entry whose child ran raddle, a raddle entry whose child ran motif, and two empty-recipe rows under a raddle and a template default.
  - The proposal asked each row to assert "whether the run directory is gone". The closure is now read-only, so each row asserts the result and that the run is left in place.
- **F3 (LOW), fixed as proposed, then extended by a self-check finding.**
  - `finalizeEntered`, `finalizeEnteredRead` and `withoutFinalizeEntered` pass `how`, and a pair at Raddle-File counts as entered only for `endedDone`.
  - New reconcile row: a pair at Raddle-File whose origin removed the entry is ended as removed, with one teardown call.
  - The `finalizeEntered` godoc and the `fabricengine` `doc.go` paragraph now say the exemption covers a done entry only.
  - Extension, see the self-check below: `readEndedSet` reads a pruned run as done when the newest commit holding it marked it ended. Rows were added to the `EndedSet` test and to `TestReconcileOrigin_Ended`.
- **F4 (LOW), fixed as proposed.**
  - `startClearsMotifRun` judges the recipe through `boardRecipe` over `boardengine.OpenHub`, so `DobbyTasks` and the guard share one resolution.
  - Rows: a dobby row for an empty recipe under a raddle default, one under the template default, and the guard test's empty-recipe rows.
- **F5 (LOW), fixed as proposed.** `TestSketchFinish_TellsTheParentOnlyOfARefusedSketch` has a `notifyErr` field and a row whose notice fails, expecting `Finish` to return nil after the one attempted notice.
- **F6 (LOW), fixed as proposed.** The `DobbyTasks` godoc and `dobbyengine/doc.go` state the exception for a done motif run of a task whose recipe the board resolves to raddle.
- **F7 (LOW), fixed as proposed.** `loomcli/doc.go` says a raddle run at Raddle-File whose entry origin shows done is not read as ended.
- **F8 (LOW), fixed as proposed.**
  - `sketchRefusedNotice` spells the Sketch `lym board upsert` payload with the slug filled in, mirroring the fields `sketchOf` sets, and tells the parent to list the entry's non-type labels after `sketch`.
  - The test asserts the payload's opening fragment.

## Gate repair

The gate found the doc comment of `TestClaimStart_RemovesTheDoneMotifRunOnlyOnceTheOwnerCheckPasses` in `internal/battencli/startguard_test.go` wrapped at a fixed column, so no test ran.
It now holds one sentence per line, with no change to its wording; no other file changed.

## Disputed

No finding of the review is disputed. Two fork findings of the self-check are, both on the ground `premise false`:

- fix-affected-set, `internal/landingshed/doc.go:150`: the raddle recipe has no Parent-Align row, so the Raddle-File exemption never reaches the rule that line states (`internal/loomrecipe/doc.go:31`, `Record-Write → Raddle-Write → Raddle-File`).
- fix-affected-set, `internal/battencli/cleanup.go:524`: the cleanup help text already reads "and by the start of a raddle run under its slug".

## Deferred

None.

## Self-check

The four forks all returned a report, saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency, LOW: `dobbyengine/doc.go:109` and the `run` help text in `cli.go` still said "recipe raddle" — fixed, both now say the board resolves the recipe to raddle.
- fix-proposal, MEDIUM, F3: a done entry origin later prunes reads as removed, which lost the Raddle-File exemption — **fixed with my own change**: `readEndedSet` reads a pruned run as done when the newest commit holding it marked it ended.
  - The reviewer's proposed `how`-keyed exemption is kept, but a done-then-pruned entry is now reported as done.
  - `battencli/cancel.go` and `resumereconcile.go`, the two readers of `How`, then treat it like any landed entry.
  - Rows: one in the `EndedSet` test and one in `TestReconcileOrigin_Ended`.
- fix-proposal, LOW, F1: `ClearMotifRun` did not re-check the entry after the owner check's pull — fixed: `claimStart` asks `ClearsMotifRun` again before removing, with a `turnedAway` row.
- fix-proposal, LOW, F8: the payload was not equivalent to `MakeSketch` — fixed: it now mirrors `sketchOf` field by field and carries the label instruction.
  - No test applies the payload to a board; its fields are in `UpsertPayloadKeys`, and a note with a leftover `notify_on_land` would have been refused by `normalizeTask`, which is why it clears it.
- fix-proposal, LOW, F4: the board config loaded per tick for every done released task, and a persistent failure warned every tick — fixed: a task's own recipe field decides without opening the board, and the warning goes through a `logger.EpisodeSet`.
- fix-affected-set, LOW: `internal/battenshed/doc.go:60` states the ended-origin rule with no Raddle-File exemption — fixed.
- fix-affected-set, LOW: `internal/landingshed/doc.go:150` is the same rule at Parent-Align — **disputed, premise false**: the raddle recipe has no Parent-Align row (`internal/loomrecipe/doc.go:31`, `Record-Write → Raddle-Write → Raddle-File`), so the exemption never reaches that row.
- fix-affected-set, LOW: `docs/overview.md:713` names only `lym beam create` as a clearer of a done motif run — fixed.
- fix-affected-set, LOW: `internal/battencli/cleanup.go:524` — **disputed, premise false**: the text already reads "and by the start of a raddle run under its slug".
- fix-affected-set left F2 and F5 unchecked, since their premises are single missing tests.
