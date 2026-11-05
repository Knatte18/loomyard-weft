# Card 10 — beam-await-entry

**Edit:**
- `internal/pawlengine#Deps`
- `internal/pawlengine#NewBeamAwait`
- `internal/pawlengine#BeamAwait.Call`
- `internal/pawlengine#BeamAwait.interruptedLanding`
- `internal/pawlengine#awaitRun`
- `internal/pawlengine#awaitRun.wake`
- `internal/pawlengine#awaitRun.enter`
- `internal/pawlengine/doc.go`
- `internal/pawlengine_test#TestBeamAwait`
- `internal/pawlengine_test#TestNewProducers_RefuseAnUnsetSeam`
- `internal/pawlengine_test#world.deps`
- `internal/shedrecipe#requirePawl`
- `internal/shedrecipe#beamAwaitEntry`
- `internal/shedrecipe#validPawlDeps`
- `internal/shedrecipe/doc.go`
- `internal/loomcli#pawlDeps`
- `internal/testkit/envkit#FullEnv`

**Create:**
- `plan:internal/pawlengine#awaitRun.recheckStanding` -> `func (r *awaitRun) recheckStanding(ctx context.Context) (awaitWake, error)`

**Uses:**
- `internal/pawlengine#awaitRun.landChecks`
- `internal/pawlengine#awaitRun.refuse`
- `internal/pawlengine#ReadyFor`
- `plan:internal/shedrun#WriteWakeStamp`
- `plan:internal/shedrun#WakeRecord`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Beam-Await's entry settles a landing it finds already accepted, with no land request, by itself (D11, and D10's goto bound), and Beam-Await stamps each completed wake for card 9's driver rule.

- `Deps` gains `ReadyStands func() (bool, error)`, whether the tracked ready record names this beam, and `StampWake func(at time.Time) error`, which records a completed wake; `NewBeamAwait` and `beamAwaitEntry`/`requirePawl` refuse either unset, naming the field.
- `BeamAwait.Call` calls `StampWake(Now())` after each wake that returns with no error and does not finish the row; a failed stamp is logged only.
- The entry, read before Beam-Await writes its own status, finds an accepted landing in either of two states, with no land request standing:
  - the board status names the hold, Finalize or a retired row name (card 5): the crash between the accept's removals and Done;
  - the ready record stands and the await marker is absent: the accept's Done removed the marker, so a later return to Beam-Await, an operator's `lym loom goto Beam-Await` from a halted Finalize, finds it. The board cannot tell this case, since every shed transition, the goto's and the resume's included, rewrites the board status through the run's `CommitStatus` seam (`internal/loomcli/wiring.go` `newCommitStatusSeam`), so it names Beam-Await before the row runs.
- On either, `awaitRun.wake` reads `ReadyStands` before `enter` writes any status:
  - The record stands: `recheckStanding` runs the accept's re-check itself, with no land request: it writes the hold naming Finalize, then `landChecks` (dobby-stop marker, dobby stop, `Recheck`), then Done after removing the dobby-stop and await markers when it passes, or `refuse` with the blockers when it fails, which lifts the hold, restarts the dobby and posts the blockers, as the accept does. So the run lands with no land request and the dobby is never asked to write one again (Constraints: its land episode stays recorded).
  - The record does not stand (only reachable by the board hold): `enter` lifts the hold by writing Beam-Await, removes the dobby-stop marker and starts the dobby through `StartDobby` whether or not the marker stood, then posts a notice that the landing was interrupted, its hold is lifted and its dobby restarted, naming `lym beam ready <slug>` to land once the operator is satisfied; a failed start is named in the notice with `lym beam start <slug>`.
  - A `ReadyStands` error decides nothing this wake and is logged; the next wake retries.
- An ordinary re-entry while the beam waits finds the await marker standing and takes today's path, so a standing ready record with members still working is judged by the dobby's land request as today.
- `pawlDeps` fills `ReadyStands` with `pawlengine.ReadyFor(loomengine.BeamReadyPath(location), loomengine.BeamReadyLockPath(location), slug)` and `StampWake` with `shedrun.WriteWakeStamp(shedrun.WakeRecord(location, shedrun.SelfRunID), NameBeamAwait, at)`; `FullEnv` and `validPawlDeps` fill both.
- Reaching inputs: a status naming Finalize (or Parent-Align) with no land request file and the ready record standing reaches `recheckStanding`; the same with no ready record reaches the lifted-hold branch; a board status naming Beam-Await, no land request, no await marker and the ready record standing, the state a goto from a halted Finalize leaves, reaches `recheckStanding`.

Tests (tier 0): `TestBeamAwait` gains rows: hold named Finalize, no request, ready standing and `Recheck` passing: Done, status never written as Beam-Await, dobby stopped, markers removed; the same with `Recheck` failing: status back at Beam-Await, dobby started, blockers posted; hold named PR-Review with ready standing: same as Finalize; a goto return (board at Beam-Await, no await marker, ready standing, `Recheck` passing): Done with no land request; a re-entry with the await marker standing and the ready record standing: no re-check, the row waits; hold with no ready record and no dobby-stop marker: status Beam-Await, dobby started, notice naming `lym beam ready`; a `ReadyStands` error: nothing written, retried at the next wake; each completed wake calls `StampWake`, and a failing stamp changes nothing. `TestNewProducers_RefuseAnUnsetSeam` gains the two fields.

**ImpactSummary:** Beam-Await gains two required seams filled by loomcli's pawl wiring and the test fixtures, and its entry settles an accepted landing it finds instead of waiting for a land request; no other producer changes.

**Commit:** 10: Beam-Await settles an accepted landing it finds on entry
