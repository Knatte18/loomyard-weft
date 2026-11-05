# Card 17 — beam-start-in-landing

**Create:**
- `internal/orchcli#ownLandingHold`
- `internal/orchcli#TestOwnLandingHold`
- `internal/orchcli#TestStart_OwnLandingStartsTheOrchAlone`

**Edit:**
- `internal/orchcli#beamFacts`
- `internal/orchcli#readBeamFacts`
- `internal/orchcli#beamUpRefusal` -> `func beamUpRefusal(slug, here, owner string, moveWayForward string, held, ownLanding bool, status string) error`
- `internal/orchcli#orchCLI.beamUp`
- `internal/orchcli#TestBeamUpRefusal`
- `internal/orchcli#TestStart_Refusals`
- `internal/orchcli/doc.go`
- `internal/beamcli#startCmd`
- `internal/beamcli/doc.go`

**Uses:**
- `internal/orchcli#orchCLI.startDobbyStep`
- `internal/boardengine#RunHolds`
- `internal/boardengine#RunStatusProducer`
- `internal/shedrun#StatusFile`
- `internal/shedrun#StatusLock`

**Intent:** `lym beam start` on a beam its own pawl run holds brings back the beam's orch and leaves its dobby down (D12), so the operator can follow a halted Finalize with the orch.

- `ownLandingHold` reports whether a held beam's hold is its own pawl run's: the beam pair's own run status is found, not done, and its current producer is the producer half of the board status. That covers Finalize and Beam-Down, and a halt at a retired row an older binary left, before card 1's move lifts it; a hold whose producer the pair's own run does not stand at is held by something else.
- `readBeamFacts` sets the new `beamFacts.ownLanding` for a held beam (`RunHolds`) through `ownLandingHold`, reading the pair's own run status at `shedrun.StatusFile(location, shedrun.SelfRunID)` under `shedrun.StatusLock`; an absent status reads as not found, and a failed read is the facts' error.
- `beamUpRefusal` gains `ownLanding`: a held beam whose hold is its own landing is no refusal; a beam held by anything else keeps today's refusal naming `lym batten status <slug>`. Ownership and anchor-change refusals come first, as today.
- `orchCLI.beamUp` with `ownLanding` skips `startDobbyStep` and starts the orch, adding the report warning that the dobby stays down while the landing runs.
- `startCmd`'s help says a beam in its own landing gets its orch back and its dobby stays down, and a beam held otherwise is refused.
- Bound: `lym beam start` never starts the dobby of a held beam; Beam-Down still takes the orch down; the orch's presence changes nothing Finalize reads.
- Reaching input: a beam whose entry status is `running · Finalize` or `blocked · Finalize` while its pair's own run stands at Finalize.

Tests (tier 0): the new `TestOwnLandingHold` covers a board status at Finalize and at Beam-Down with the pair's run there (own), a board status at Finalize with the run done or absent (not own), and a board status naming a row the pair's run does not stand at (not own). `TestBeamUpRefusal` gains rows: held by its own landing passes; held otherwise still refuses naming `lym batten status`. The new `TestStart_OwnLandingStartsTheOrchAlone` drives `newStartHarness` with `readBeamFacts` answering a held beam in its own landing and asserts the orch started, no `dobby` call recorded and the dobby-stays-down warning reported. `TestStart_Refusals` keeps its held-beam row refusing with `ownLanding` false.

**ImpactSummary:** one re-signed pure refusal with one caller in orchcli and its tests, one new fact read from the beam pair's own run status, and one new pure helper; every other bring-up is unchanged.

**Commit:** 17: lym beam start brings back the orch of a beam in landing
