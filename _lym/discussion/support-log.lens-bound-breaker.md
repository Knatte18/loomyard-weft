```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break each claim and bound in the decision record with a concrete path.

**Findings**

1. **MEDIUM. D3/D4: an escalate halt loses its protection from re-steps on a new build. Nothing in the record says so, so the "arbiter first" claim does not hold.**
   - **The two re-step paths:**
     - In the driver's rule table, the hand-over row ("a stop the on-disk build could change … hand over to a fresh driver") sits above the `JudgesFirst` row. `binaryEligible` (`sheddriver/driverules.go:181`) is true for every blocked stop.
     - Batten's child `HandBack` seam (`battencli/wire.go:1067`, "so the binary re-step leaves it to the orch") reads `arbiterstop.HandsBack`.
   - **Why this is safe today:** the hand-back row comes first and protects every landing halt from both paths.
   - **Counterexample under D3:**
     1. Finalize halts on a fixer `escalate`.
     2. The operator deploys a new `lym`, and dobby survives the deploy.
     3. The driver hands over once for the new build.
     4. The fresh driver starts at the blocked stop. Its only start check is `finalizeGuard`, which tests nothing but parent dirt, so it re-steps Finalize.
     5. The gate fails again, and the fixer runs again on its returned attempt with no new decision recorded.
     6. The fixer then either escalates again, at the cost of one more fixer session, or "fixes" the collision itself. In the second case nobody judges the collision: not the arbiter, not the orch.
   - Any arbiter attempt still open on the old stop key is orphaned.
   - **Fix:** either say in D4 that this is accepted, as it is for Webster today, and state its bound (once per build), or keep the escalate stop marked hand-back for the binary re-step paths while the arbiter-first row decides it.

2. **LOW. Auto-mode assumptions: the reason given for keeping `Overlap` is wrong.**
   - The record says "the finalize-mode fixer and later Parent-Align read the record."
   - No Go code reads `MergeRecord.Overlap`. Only `parentalign.go:490` and Finalize write it.
   - Parent-Align reads the earlier record for `AlignedTip` and `MergedTip`, not for the overlap.
   - The fixer stencil reads the whole merge record file (`landing-template-fixer.md:20`), so the field reaches the fixer only as prompt text.
   - **Fix:** reword the reason to "the fixer reads the merge record file, overlap included". The decision itself can stay.

3. **LOW. D2: a darn run in flight loses its review input when the binary is upgraded.**
   - A darn run that an older build already rerouted to Code-Bouncer has an overlap note newer than the perch's settle.
   - Once `ReviewInput` is nil, `keptReviewInput` (`shedadapters/amendment.go:58`) returns "", so the re-entered perch reviews without the note.
   - The record says nothing about this, though it does address the older-escalate case.
   - **Fix:** one line under Constraints accepting it.

**Claims attacked**
- D1, the retry merges, gates and lands: held. There is still only one retry under the same lock.
- D1 bound, a spent cap or over-cap change still reroutes or halts: held (`keepFixerCommits` and `RerouteReviewed` are unchanged).
- D1 bound, the parent-lock wait under the conflict session: held. Darn and pawl already run the session under the lock.
- D2, `ReviewInput` nil is the absent value: held. `entries_bouncer.go` passes nil, and `keptReviewInput` returns "" for nil.
- D3, `HandsBack` is one predicate shared by the envelope mark (`arm.go:309`), the driver (`drivewiring.go:302`) and `arbiterstop.Check`: held. It is also batten's binary re-step seam, which is finding 1.
- D3, every other landing halt keeps `LandingStopPrefix`: held. That covers the fixer `haltReason` cases and the lock re-check in `parentguard.go`.
- D4, a resume with the halted row as goto keeps the row entry: held. The goto appends `HistoryEntry{Producer: target}` (`shedengine/goto.go:135`), so `rowEntry` keeps the same attempt record and Parent-Align keeps its fork point.
- D4, the escalate turn was given back: held (`Spent--` in `fixerrun.go`).
- D4, the per-run spawn cap stops a loop of escalate and resume: held. Every resume grows the history, so each escalate is a new stop key and spends a spawn from `MaxSpawns`.
- D4, an arbiter that is off or out of spawns hands back: held. `HoldReading` reports `On` and `SpawnsLeft` to batten.
- D4, the driver passes the reason: held. `Stop.Reason` is set for both envelope stops and status stops.
- D4, the arbiter is "first": broken by finding 1.
- The constraint on pawl: held (`LargeMergeLands` and `ConflictReroutes` are false for pawl today, and its `RerouteReviewed` is nil).
- The constraint on older escalate halts: held. They keep `LandingStopPrefix`.

**Tool calls**
1. `arbiterstop/holdreading.go` and a grep of `HandsBack` users: needed to see batten's hold and every reader of the hand-back predicate.
2. `battencli/wire.go` around the `HandBack` seam and the `Stop.Reason` lines in `drive.go`: needed to confirm batten's re-step reads the predicate and where the driver gets the reason.
3. The history appends in `shedengine` and `binaryEligible`: needed for the goto row-entry claim and the hand-over path.
4. `ReviewInput` users and the fixer stencil's mentions of the merge record or overlap: needed for D2's nil claim and the `Overlap` reader claim.
5. `entries_bouncer.go`, `amendment.go` and `envkit.go` excerpts: needed to confirm how a nil `ReviewInput` is handled.
