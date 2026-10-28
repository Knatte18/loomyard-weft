```yaml
lens: code-correctness
outcome: reported
```

**Per-function list**

- `Finalize.mergeParent`, a conflicting merge: `mergeIntoTask` calls `resolver.Resolve`, which runs the conflict session under the lock already held. It returns `parentMerge{upToDate: false}`. `gate.clean`, `judgeParentMerge` and the merge-record write follow, and the call returns `done=false`, so the gate runs. Correct.
- `Finalize.mergeParent`, an overlapping-file merge: `judgeParentMerge` fills `overlap`, and `MergeRecord.Overlap` is still written. The switch and the overlap note are gone, so the call goes on to the gate. Correct.
- `Finalize.mergeParent`, an up-to-date merge: `upToDate` is true, so `mergeCommit` stays empty and the record gets `UpToDate: true`. Correct.
- `Finalize.mergeIntoTask`, a `Resolve` error: it is wrapped, and `mergeParent` returns `cancelErr` first when the context is cancelled. Correct.
- `Finalize.mergeIntoTask`, `OutcomeStuck`: the reason is returned and `mergeParent` calls `stuckOrCancelled`. Correct.
- `Finalize.mergeIntoTask`, the removed counting wrapper: `deps.Shuttle` is now passed to `mergeresolve.New` directly. A nil shuttle stays nil, so the constructor's refusal of nil is preserved. Correct.
- `Finalize.Call` retry: after `ErrMergeInRequired` it re-runs `mergeAndGate`, so a code-side parent commit is merged and gated. There is no reroute path left in `mergeParent`. Correct.
- `Finalize.reroute` and `rerouteReason`: still used by the over-cap and spent-cap path at `finalize.go:743`, so they are not dead. Correct.
- `fixerOutcome.haltReason(row, escalateJudgedFirst)`, escalate with true: the prefix is `EscalateStopPrefix` ("landing escalate: ") and the way forward names `lym loom decision add`. Correct.
- `fixerOutcome.haltReason`, escalate with false (pawl): the prefix is `LandingStopPrefix` and the way forward names no decision verb. Correct.
- `fixerOutcome.haltReason`, spent, timeout, no-report, dirty and undecodable ends: `prefix` stays `LandingStopPrefix`. The `default` case returns "" as before. Correct.
- `ParentAlign.haltSpent`: it passes `EscalateJudgedFirst`, but the end is `fixerEndSpent`, so the flag is ignored. Correct.
- `ParentAlign.align`: the escalate halt carries the flag. Correct.
- `landingshed.HandsBack`: "landing escalate: " does not start with "landing stop: " or the reroute-cap prefix, so it is false. Correct.
- `landingshed.EscalateHalt`: true for Parent-Align, Finalize and the retired Merge-In with the prefix, false for any other producer or reason. Correct.
- `sheddriver` `facts`, a stop with `FromStatus` false: the reason is `stop.Reason`. Correct.
- `sheddriver` `facts`, a stop with `FromStatus` true: the reason is `stop.Error`, which holds the persisted halt text. Correct.
- `sheddriver` `facts`, nil `JudgesFirst` or nil `Arbiter`: guarded by the existing condition. Correct.
- `loomArbiterSeam.JudgesFirst`: Webster is true, a landing escalate is true, and a Finalize stop with `LandingStopPrefix` is false and meets the tackler. Correct.
- `Spec.restepHolds` and `RestepHold`: nil gives false, a non-blocked state gives false, and it is computed beside `hand_back` in the loop envelope (`loop.go:336`) and in `status.go`. Correct.
- `loomCLI.specFor`: `RestepHold` is set only for the loom and darn recipes, so pawl stays nil and `restep_hold` is false. Correct.
- `battencli.restepSkips`: a non-blocked status gives false. A blocked landing halt or decision halt gives true (as before), a blocked escalate halt gives true (new), and anything else gives false (as before). Correct.
- `landingDeps`: `EscalateJudgedFirst` is true for loom and darn and false for pawl. Correct.
- `darnwiring`: the `ReviewInput` override is removed, so darn falls back to the shared wiring's fixer-report seam, as D2 asks.

**Findings**

None. No wrong result, swallowed error or inconsistent failure-path state turned up in the changed code.

Tool calls:
1. Read `finalize.go` 786-860 (`mergeParent`'s tail) and grepped the `reroute` helpers: my context held only the diff, not the surviving lines.
2. Read `finalize.go` 924-945 (`rerouteReason` and `reroute`): to confirm the helpers are still used and their wording is not stale.
