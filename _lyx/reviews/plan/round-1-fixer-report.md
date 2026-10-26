# Fixer report — plan, round 1

Every finding is fixed; one self-check finding is deferred and one disputed.
`lyx loom validate-plan` reports `ok: true` after the changes, with informational findings only.

## Changes

- **F1** (fixed as proposed): card 3 adds `TestFabricReplay`, `TestRunLock` and `stepOwners` to its Edit list, and its fixture bullet says how each moves its task owner.
  `TestRunLock`'s row moves the owner onto the prime, recording `box` as the prime owner, and splices no `owner_machine` into the held task.
  Spliced into the task, that key would be dropped at load, so the row's "written file still carries it" check would fail.
  Card 25 adds `TestRefuseStartOwnedElsewhere`, whose three rows assert the fixture's beam.
- **F2** (fixed as proposed): card 25 adds `cancelDeps`, `hubCancelDeps` and `cancelFake.deps`.
  `cancelDeps.Owner` now returns the adopt subject, read through `Owners.AdoptSubject` with `hubgeom.ParentOf`, and `judgeOrigin`'s owner-read branch passes it on.
  A `TestCancelRun` row covers an unreachable origin for a task whose beam another machine owns.
- **F3** (fixed as proposed): card 2 adds `pattern/PATTERN-run-process-owner.md` and rewrites its `End`-only lines, the rule line and the review line, to name `End`, `EndSparing` and `EndRole`.
  The `PATTERN.md` one-liner is deferred; see below.
- **F4** (fixed as proposed): re-sign arrows on `namedEntries` (card 4), `beamMembers` (card 13), and `readOriginOwners` and `classifyEntries` (card 25).
  `ReconcileOrigin` joins card 25, and the Intents say what the new parameters carry.
- **F5** (fixed as proposed): card 4 adds `refuseDropLocalAnchorChange`.
  Its way forward re-runs the same `lyx adopt` with `--drop-local` and never names the moved slug.
  Card 4 also adds `fabricengine_test#TestDropLocal`, which the review did not name: its rows at `droplocal_integration_test.go:112,119` assert `lyx adopt task --drop-local` and would break at card 4.
- **F6** (fixed as proposed): card 14's `landHaltReason` names the cancel, then the board move, in one halt, for a member with a pair or a run in flight; `TestFinalize_LandCheck`'s rows assert both.
- **F7** (fixed differently): card 15 gains a `DobbyLive` seam, which card 17 wires to `dobbyengine.Live` over `hubgeom.DobbyGeometry`, and a `TestBeamAwait` row for a re-entry with the dobby down.
  - **What changed from the proposal:** the proposal checked the dobby "when Beam-Await is entered with no request standing" and posted the interrupted-landing notice. That also fires on the first entry from Beam-Up, whose freshly spawned dobby may not hold its lock yet (self-check, fix-proposal).
  - **First attempt, rejected:** telling the two entries apart by `EntryProducer` does not work. Shed's status commit (`internal/loomcli/wiring.go:200-216`) rewrites the board status to Beam-Await before the row runs.
  - **The fix:** a durable await marker under `.lyx/beam/`, `AwaitMarkerName`. Beam-Up removes it, Beam-Await writes it on entry, and the Done path removes it. Only an entry that finds it standing reads `DobbyLive`, and the notice says the beam's dobby is down, naming `lyx beam start` and `lyx beam land`.
  - **Knock-on edits:** card 17's `PawlPaths` and its test sentence, and the overview's shared decision on `.lyx/beam/`, now include the marker.
- **F8** (fixed as proposed): card 20's `refuseBeamCancel` words member blockers as `landHaltReason` does once `RunHolds` reads the beam held, and keeps `BeamLandRefusal`'s text otherwise.
  It reads the hold through two new `cancelDeps` fields, `LandingHeld` and `LandBlockers`.
  A `TestCancelRun` row covers a beam at Parent-Align with a member.
- **F9** (fixed as proposed): card 11's slug-with-`--all` refusal names dropping either one, and its test row asserts both.
- **F10** (fixed as proposed): card 1's `checkBeamPayload` way forward matches `judgeCreateEntry`'s, and `TestUpsertFieldAllowlist` asserts the task slug is not named after `lyx beam create`.
- **F11** (fixed as proposed): card 25 deletes `ownedElsewhereRefusal`, drops the clause that kept it, and says its two test users build with `ownedElsewhereFor`.
- **F12** (fixed as proposed): card 16 drops `TestPawlEntries_RefuseNilSeams` and adds `simpleEntryCases` and `zeroEnvField`, with `Pawl.<Field>` rows.
  The Intent says each entry names a refused seam in that dotted form, as `websterEntry` does through `requireSeam`; `TestSimpleEntries_UnderfilledEnv` checks the error contains the field name.
- **F13** (fixed as proposed): card 18's "What it lets through" names the draft skip and its bound.
- **F14** (fixed as proposed): card 7 drops the refusal of a named slug that is no local beam pair, and its row, and says why no caller reaches it.
- **F15** (fixed as proposed): card 7's failure way forward names `lyx beam start root` for a prime parent.
  It is asserted in `TestStartFirstPass`'s `FailedDobbySpawnWarnsAndKeepsThePass` row, whose parent is the prime; the test has no row named "prime row".
- **F16** (fixed as proposed): card 16 adds `internal/runprofile/doc.go`, naming the `pawl` block and default.
- **F17** (fixed as proposed): card 9 adds `TestPrimeSegments`, card 10 adds `TestCLI_OrchRunnerReadsNoPause` and rewords the `startdobbies_test.go` header, and card 1 drops `destroy.go`'s `ResetRecordsToParent` header clause, as text only.

## Self-check

- fix-claims, LOW, card 18's bound left out adopt's `ResumeRun`: fixed; the bound now names it.
- fix-consistency, MEDIUM, the `PATTERN.md` one-liner still says "ended only via `runproc.End`": deferred, see below.
  The fixer's own rewrite of the background's opening line was reverted, so the opening still matches `PATTERN.md`.
- fix-consistency, MEDIUM, card 25 missed `cancelFake`: fixed; it is on the Edit list and gains a subject field.
- fix-consistency, LOW, card 20's opening and field list were stale: fixed; the opening names the held wording, and the field bullet covers the new fields and a nil `LandingHeld`.
- fix-proposal, MEDIUM, F7's check posts a false notice on a fresh run: fixed differently from the suggested grace period. A grace period is a timing race; the fix is the await-marker discriminator described under F7.
- fix-proposal, LOW, card 20 named a cancel for every member: fixed; a member with no run status gets the board move alone.
- fix-affected-set, MEDIUM, card 25 missed `TestArmAt_RunOwnedElsewhereWritesNoSeed`: fixed; it is on the card and asserts `lyx adopt root`.
- fix-affected-set, MEDIUM, card 3 missed `pairteardown#TestTeardown_OriginSeams`: fixed; it is on card 3, with `mine` and `theirs` seeded as beam projects that carry their owners.
- fix-affected-set, LOW, `TestUpsertFieldAllowlist`'s owner subtest: disputed (premise false); see below.
- fix-affected-set, LOW, `smoke_cycle_test.go`'s header still names `lyx orch start`: fixed; card 10 rewords it.

## Disputed

- **Self-check fix-affected-set LOW, `TestUpsertFieldAllowlist`.** Ground: premise false.
  The subtest at `internal/boardengine/store_test.go:486-526` writes `owner_machine` on task `t1` under prime owner `neve`.
  Its accepted row writes `neve`, which is still `t1`'s effective owner through root under card 3, so the write is still a no-op.
  Its refused row writes `box` and asserts only `lyx adopt` and `"neve"`. Card 3's refusal names `lyx adopt root` with owner `neve`, so it still contains both.
  No row reads back a stored task owner.

## Deferred

- **Self-check fix-consistency MEDIUM, `PATTERN.md`'s `PATTERN-run-process-owner` one-liner.** It needs a declared PATTERN change, and the decision record does not have one.
  Declaring it in the overview's `## PATTERN changes` failed `lyx loom validate-plan` with blocking `pattern-changes-mismatch`: "the plan's PATTERN changes section declares … `PATTERN-run-process-owner` … and the decision record lacks it".
  The decision record is outside this fixer's write surface.
  Adding that PATTERN change, which D4's constraint already permits in substance, is for the decision record's owner. The plan can then declare the line, and card 2 can add `PATTERN.md`.

## Observed, not changed

- Card 10 has `TestSmokeOrch_OneFullCycle` start its orch through `BeamUp` in-process, but its file header says it drives the built binary because the start spawns `lyx orch watch` from `os.Executable()`.
  The fix only renamed the verb in that header; whether the in-process start meets the header's invariant is text this round did not touch.
