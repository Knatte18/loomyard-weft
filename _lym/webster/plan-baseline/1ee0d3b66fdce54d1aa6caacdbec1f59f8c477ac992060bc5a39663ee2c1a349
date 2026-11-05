# Card 9 — await-survives-loop-exit

**Edit:**
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed#TestInterruptPolicyFor`
- `internal/loomrecipe#TestInterruptPolicies_MatchAssembledRows`
- `internal/loomrecipe#TestInterruptPolicies_MatchEngineIdentity`
- `internal/sheddriver#Deps`
- `internal/sheddriver#goDriver.facts`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/sheddriver/doc.go`
- `internal/shedrun#TestPathConstructors`
- `internal/loomcli#loomCLI.goDriverDepsFor`

**Create:**
- `internal/sheddriver#wakeRestartGap`
- `internal/sheddriver#lastRestepAt`
- `internal/shedrun#WakeRecord`
- `internal/shedrun#WriteWakeStamp`
- `internal/shedrun#ReadWakeStamp`
- `internal/shedrun#TestWakeStamp`
- `internal/arbiterstop#TestEligibility_InterruptedPawlRows`

**Extends:**
- `internal/loomshed/doc.go`
- `internal/shedrun/doc.go`

**Uses:**
- `internal/sheddriver#repairCount`
- `internal/sheddriver#decideDrive`
- `internal/sheddriver#driveRepairCap`
- `internal/arbiterstop#Eligibility`
- `internal/arbiterstop/doc.go`
- `internal/pawlengine#NameBeamAwait`
- `internal/pawlengine#NameBeamUp`
- `internal/pawlengine#NameBeamDown`

**Intent:** A loop exit at Beam-Await is resumed with no hand `lym loom resume`, however long the beam waits (D11's driver half).

- `loomshed.InterruptPolicies` gains `pawlengine.NameBeamUp`, `NameBeamAwait` and `NameBeamDown`, each `shedverbs.InterruptPolicyReinvoke`, with a comment: each spawns no agent and is idempotent. The driver's existing reinvoke row (`decideDrive`) then re-steps an interrupted pawl stop, and `arbiterstop.Check`, which reads `loomshed.InterruptPolicyFor`, makes one past the repair cap eligible for the arbiter with no change to `arbiterstop.Eligibility`.
- `shedrun.WakeRecord` is `wake.json` in the run's `ScratchDir`, never tracked; `WriteWakeStamp` writes `{"row":..,"at":..}` atomically, creating the directory, and `ReadWakeStamp` returns the stamp when the record names row, not found for an absent file or another row. Card 10's Beam-Await writes it after each completed wake.
- `sheddriver.Deps` gains `CompletedWake func(row string) (time.Time, bool, error)`, the recipe wiring's read of a row's last completed-wake stamp; nil reads none. `goDriver.facts`, after `repairCount`, restarts `RepairCount` at zero when `CompletedWake` returns a stamp for the stop key's row that is at least `wakeRestartGap` later than `lastRestepAt`, the `At` of the key's last restep settle record, exactly as D11 states; otherwise the count stays every restep record of the key. With no restep record the count is already zero. A `CompletedWake` error is logged and changes nothing. The rule reads no row or recipe name, so it stays recipe-agnostic.
- `loomCLI.goDriverDepsFor` fills `CompletedWake` for a beam run with `shedrun.ReadWakeStamp(shedrun.WakeRecord(location, runID), row)`, and leaves it nil otherwise.
- Bound: a loop that dies within ten minutes of each re-step is re-stepped at most `driveRepairCap` times and then reaches the tackler and the arbiter; one that lives longer is re-stepped again after each such life, which is at most the cap's times per ten minutes.

Tests (tier 0): `TestInterruptPolicyFor` gains the three pawl rows; `TestInterruptPolicies_MatchAssembledRows` and `TestInterruptPolicies_MatchEngineIdentity` include the pawl recipe's rows. The new `TestEligibility_InterruptedPawlRows` asserts an interrupted stop at each of Beam-Up, Beam-Await and Beam-Down, its policy read through `loomshed.InterruptPolicyFor`, is eligible under `Eligibility`. `TestGoDriver_Settle` gains rows: an interrupted Beam-Await stop with two restep records and no stamp goes to the tackler; with a stamp eleven minutes past the last restep it is re-stepped again; a second crash right after that re-step, its stamp now older than the new restep record, goes to the tackler; with a stamp five minutes past the last restep it goes to the tackler; a `CompletedWake` error leaves the count as read. `TestPathConstructors` gains `WakeRecord` under the scratch directory. The new `TestWakeStamp` covers a round trip reading the row's time, another row read as not found, and an absent file read as not found.

**ImpactSummary:** three new interrupt-policy entries, one optional driver seam filled only for a beam run, and a scratch wake record; every other recipe's driver sees no change.

**Commit:** 9: Re-step an interrupted Beam-Await stop after each completed wake
