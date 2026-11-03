# Card 13 — remove-follow-up

**Create:**
- `plan:internal/pairteardown#BattenFollowUp` -> `func BattenFollowUp(slug string, record shedengine.Status, found, ownerHeld bool) string`
- `plan:internal/pairteardown#TestBattenFollowUp` -> `func TestBattenFollowUp(t *testing.T)`

**Edit:**
- `internal/pairteardown#Result`
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.Run`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#TestTeardown_CallSequence`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#TestRemoveFields`
- `internal/tendercli#EndOwnedElsewhere`
- `internal/tendercli#ownedElsewhereEnder`
- `internal/tendercli#ownedElsewhereEnder.run`
- `internal/tendercli#TestOwnedElsewhereEnder`
- `internal/landingshed#TestFinalize_MergeMarkPushOrderAndVerdict`

**Extends:**
- `internal/pairteardown/doc.go`
- `internal/fabriccli/doc.go`
- `internal/tendercli/doc.go`

**Uses:**
- `internal/battenrecipe#NameLand`
- `internal/battenrecipe#PastLand`

**Intent:** Decision record D3: a way forward naming `lym fabric remove <slug>` for a slug whose batten run record stands also names the step that settles that record, and `lym fabric remove` warns after removing such a pair.

`BattenFollowUp` words the follow-up from the prime's batten record of the slug:
- No record, a `done` record, a record at Worktree-Teardown (`battenrecipe.PastLand`), or `ownerHeld` (an owner pause covers the slug): `""`.
- A record before Land: `then run \`lym batten cancel <slug> --remove\`, which clears the run's record and the task branch the next run's add would refuse`.
- A record at Land (`battenrecipe.NameLand`): `the run's work already reached its parent, so never cancel it: run \`lym batten run <slug>\`, whose failed stop names the restore of each branch the removal took, run those steps, then \`lym batten run <slug>\` again` — D2's Land way forward, which that pass computes from the tags the removal left.

Wiring:
- `Teardown` gains `battenRecord func(slug string) (shedengine.Status, bool, error)`, reading the prime's `shedrun.StatusFile(prime, slug)`, and `ownerHeld func(slug string) (bool, error)`, reading `pauserecord.ReadCoverage(hubgeom.PauseGeometry(prime).RecordPath, slug, now).OwnerHeld`; `New` sets both, and a nil seam reads as no record (a hand-built test `Teardown` warns nothing).
- `Result` gains `Warning string` (`warning`, omitempty). `Teardown.Run`, after a successful removal of a request without `LeaveClaim` (a batten cancel's own removal settles the record itself), sets it from `BattenFollowUp`, prefixed `the batten run of <slug> still stands; `; a read error is logged and adds no warning. No batten record is written.
- `fakeSubstrates` gains a batten-record fake and an owner-held fake feeding the two seams.
- `removeFields` adds `warning` when set; `TestRemoveFields` gains that row.
- `ownedElsewhereEnder` gains `battenRecord` and `ownerHeld`, which `EndOwnedElsewhere` fills from the prime where it builds the ender; `run`'s failed-end error, which names `lym fabric remove <slug>`, appends `BattenFollowUp`'s text when non-empty.
- Finalize's stop for origin's parent already holding the work (`adopterFirstRepair` in `internal/landingshed/finalize_owner.go`) is the exception: its text stays as today, naming `lym fabric remove <slug>` and the standing owner pause, with no cancel, since a cancel lifts the owner pause (`liftSlugPauses`) and another machine has landed the work.

Extend `internal/pairteardown/doc.go` with the warning and its wording rule, `internal/fabriccli/doc.go`'s remove paragraph with `warning`, and `internal/tendercli/doc.go` with the follow-up the owned-elsewhere ender's refusal appends.

Tests:
- `TestBattenFollowUp`, `tier0`, a table over the wording alone: no record, `done`, Worktree-Teardown and owner-held give `""`; Run-Shed and Worktree-Create name `lym batten cancel <slug> --remove`; Land names `lym batten run <slug>` and not `lym batten cancel`.
- `TestTeardown_CallSequence` gains rows driving `Teardown.Run` over `fakeSubstrates`: a batten record at Run-Shed sets `Result.Warning` naming the cancel; at Land it names `lym batten run` and no cancel; with the owner-held fake true it is empty; with `LeaveClaim` set it is empty.
- `TestOwnedElsewhereEnder` gains a row: a failed end with a batten record at Run-Shed names `lym fabric remove <slug>` and then `lym batten cancel <slug> --remove`.
- `TestFinalize_MergeMarkPushOrderAndVerdict`'s adopter-first assertions gain one that the note does not contain `lym batten cancel`.

**ImpactSummary:** Adds two optional seams and one omitempty result field to the pair-teardown composite, two seams to tender's owned-elsewhere ender and one envelope field to `lym fabric remove`; every other teardown route (batten cancel, Worktree-Teardown, hub reconcile) either sets `LeaveClaim` or reads a record past Land, so none gains a warning.

**Commit:** `13: A removal of a batten run's pair names the step that settles its record`
