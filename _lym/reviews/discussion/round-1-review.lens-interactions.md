```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state against the record's own paths, including the failure and restart paths.

**Pairs checked**
- D1 × D2, one merge path once the flags and `taskMerger` go: held. `mergeIntoTask`'s `ConflictReroutes` branch is the only user of `parentMerge.conflicts`, and the resolver is built for every recipe (finalize.go:160-181).
- D1 × D3, more Finalize gate and fixer runs, so more Finalize escalates: held. The extra escalates take D3's route.
- D1 × D4, an arbiter resume with Finalize as the goto: held. A re-entered Finalize clears the premerge, reads the merge as up to date and re-runs the gate and fixer. The escalate gave its attempt back (fixerrun.go:256-257).
- D3 × D4, the escalate prefix reaching `JudgesFirst(producer, reason)`: **broken** for a stop built from a status (finding 1).
- D3 × the driver's `HandsBack` row and D4's ordering: held. Once `HandsBack` is false, the stop falls through the budget, precondition, awaiting and hand-over rows to the `JudgesFirst` row, and D4 states that the hand-over comes first.
- D3 × `arbiterstop.HandsBack` and `DecisionHalt`: held only if the new prefix does not open with `decision halt: ` (finding 2).
- D3 × batten's re-step skip × D4's arbiter hand-on: held. The batten seam stays the only re-step reader, and after a hand-on the stop is left to the orch.
- D3 × the constraint on older `LandingStopPrefix` escalates: held. `HandsBack` still matches the old prefix, so the driver, `arbiterstop` and batten's re-step all still treat it as a hand-back.
- D4 × the escalate-resume loop × D3's given-back attempt: held. The fixer caps never spend on an escalate, and D4 names the arbiter's spawn cap as the bound on the loop.
- D4 × the `JudgesFirst` signature × batten's driver: held. Batten wires no `JudgesFirst`; only drivewiring.go:133 sets it.
- D1 × Out (the `on_reroute` edges stay for spent and over-cap) × AC 4: held.
- D1 × `PATTERN-parent-lock`: held. The conflict session runs inside Finalize's lock.
- D2 × D5, the overlap note removed and the rubric and recipe comments edited: held.
- AC 6 to 8 × D3 and D4: broken in part. No criterion covers a stop built from a status (folded into finding 1).
- AC 10 × D3's "every other halt": held. The pushed-session halt is covered by D3's catch-all.

**Findings**

1. **MEDIUM, design. D4, with D3 and AC 7.**
   D4 passes "the stop's reason, which the driver's stop already carries" to `JudgesFirst`.
   A stop built from a status carries the persisted halt text only in `Error`; its `Reason` is empty unless the run is paused (sheddriver/driverules.go:110-125).
   The driver builds such stops at its start check and on its recheck and restart paths (sheddriver/drive.go:629, 776, 781, 871).
   So a restarted or rechecked driver would not see D3's prefix: it would send the landing escalate to the tackler first, which contradicts D4.
   The other readers of this text read `Error`: arbiterstop.go:164-165, and `budgetExhausted` reads both fields (driverules.go:254).
   D4 should name the field: `Reason`, or `Error` for a stop built from a status, as the budget row does.
   AC 7 should add a restarted driver at a status-built escalate stop going to the arbiter with no tackler call.

2. **LOW, consistency. D3.**
   D3 requires the new prefix to differ from `LandingStopPrefix` only.
   `arbiterstop.HandsBack` and `DecisionHalt` also match any reason opening with `decision halt: ` (shedadapters/bouncerdecision.go:28; arbiterstop.go:62-63, 165).
   A prefix chosen with that opening would be marked a hand-back and refused by the arbiter, which undoes D4.
   D3 should also require the prefix to differ from `shedadapters.DecisionHaltPrefix`.

**Tool calls**
1. A grep for the `DecisionHaltPrefix` value and the `LandingHalt` setter in `arbiterstop`. My context held `arbiterstop.HandsBack`, but not the prefix's value or which status field `Stop` reads.
