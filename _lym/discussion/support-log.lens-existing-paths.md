```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that read or write the state D1–D4 change.

**Findings**

1. MEDIUM, D3/D4: batten's binary re-step now re-steps a blocked landing escalate, and the record says nothing about it.
   - Evidence: `internal/battenshed/innerrun_binaryrestep.go:63` skips the re-step only when `deps.HandBack(status)` is true. `internal/battencli/wire.go:1066` wires that to `arbiterstop.HandsBack`, which reads `landingshed.HandsBack`. The `battenshed/doc.go:145` contract is that only landing halts are left to the orch's resume.
   - Effect: D3 makes `HandsBack` false for an escalate. That stays true after the arbiter hands it on, because the status error keeps the escalate prefix. So any new lym build re-steps Finalize or Parent-Align while the orch or operator is still deciding the collision.
   - What the re-step does: it re-runs the gate. If the gate fails, the fixer runs again, since the escalate gave its turn back. It likely escalates again under a new stop key, which spends another arbiter spawn and may send the orch a second hand-on.
   - Fix: the record should decide this. Either the escalate predicate also stops the binary re-step after a hand-on (keyed on the arbiter's hand-on attempt), or the record states that a re-step re-runs the row and accepts that bound.
2. LOW, D3/Constraints: the hand-back callers already in place agree with D3, but the record names only the driver, the envelope and arbiterstop.
   - Evidence: `arbiterstop.HandsBack` is the only route, used by the loom driver's `HandsBack` (`drivewiring.go:302`), the loop spec's `HandBack` (`arm.go:309`, the envelope mark) and batten's re-step seam (`wire.go:1066`). D3's single change reaches all three consistently.
   - The batten consequence is finding 1. The record could name batten's re-step as a reader so the Plan doesn't treat it as untouched.

**Paths traced, per decision**

- **D1, conflict resolved under the lock in loom:**
  - The pre-merge clear (`StopConflictSession`, then the abort of a parked merge-in) already serves darn's resolver path. Survives.
  - The verify hold and pause cover reads of the parent lock are unchanged. Survives.
  - Parent-Align's later read of `AlignedTip` from the merge record, after a fixer-spent reroute: Finalize still carries `AlignedTip` forward. Survives.
- **D1, retry after a merge-in-required error:** it now gates and lands, and the record states this. Survives.
- **D2, darn `ReviewInput` set to nil:**
  - `shedadapters.keptReviewInput` treats a nil seam as no input (`amendment.go:59`).
  - A changed subject still starts an amendment generation, so the darn fixer's reroute commits are still reviewed. Survives.
  - Stale `overlap-note.md` files in the scratch dirs of in-flight darn runs: no reader is left. Harmless.
- **D2, loom `ReviewInput`** (`wiring.go:734`, the Parent-Align fixer report) is untouched. Survives.
- **D3, driver rule table:**
  - `driverules.go:101/115`: the envelope `hand_back` mark and the status-based `HandsBack` both read false for an escalate, so the stop falls through to the later rows. Survives as intended.
  - The build hand-over row comes before the JudgesFirst row: it hands over once per build, then the fresh driver reaches the JudgesFirst row. Survives.
- **D3, `arbiterstop.Check`/`Eligibility`:** a blocked stop that is neither a landing halt nor a decision halt is eligible. Survives.
- **D3, `HoldReading`** (batten's eligible-stop hold): the escalate is now eligible, so batten holds while the arbiter judges it. Survives.
- **D3, batten binary re-step:** breaks (finding 1).
- **D4, arbiter resume with goto set to the halted row:** shedengine admits a target at or before the reference row (`shedengine/doc.go:75`). Finalize's `StartCheck` dirt guard still runs. Survives.
- **D4, an arbiter re-call on the same stop key** returns the earlier verdict and spawns nothing. A changed key (after a re-step) spawns again, which ties into finding 1.

**Extra calls**

- Grepped the callers of `HandsBack`, `LandingStopPrefix` and `HandBack` outside landingshed: the context lacked the batten and arm call sites.
- Read `battencli/wire.go` around `HandBack`, `drivewiring.go`'s `HandBackDeps`, and `HoldReading`: the context lacked how batten uses the predicate.
- Grepped `battenshed` for `HandBack` and shedengine for the goto admission rule: the context lacked the re-step rule and the goto bound.
- Grepped `ReviewInput` consumers: the context lacked how a nil seam is handled.
- Read `shedadapters/amendment.go`: needed to confirm darn's reroute review survives without the overlap note.
