---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/loomcli/start.go:143
    summary: the parked record's verb literal merge-in is declared as a separate mergeInVerb constant in loomcli and in landingshed, and a third time as a bare literal where fabricengine writes the record
  - id: F2
    severity: LOW
    class: consistency
    location: internal/fabricengine/configchanges_integration_test.go:86
    summary: the row "a parent change synced into the task is not reported" asserts Base equals the parent tip, so its two diffs coincide and the row passes without the intersection
---

Review range: `status.json` gives `product.parent` = `main`, and `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD`, which is the eight card commits.
`go build`/vet were not run directly because the hook refuses raw module-wide go runs.
`lyx gate test` over `internal/landingshed`, `internal/loomcli`, `internal/agentname`, `internal/mergeresolve`, `internal/shuttleengine`, `contracts/stencils` and `cmd/lyx` passed, so the test-budget rows for `internal/landingshed` and `internal/loomcli` match their top-level counts.
`lyx gate test --tags integration ./internal/fabricengine` passed.

Verdicts on the focus questions, none of which found a defect:

- **Card 5 order and scope.**
  In both `Publish.Call` and `Finalize.mergeInStep`, `preMerge.clear` runs before the `before the merge-in` clean check, and `Finalize` reaches it on both passes.
  `conflictSessionStopper` stops only live, non-retiring strands whose role `agentname.MatchesRole` matches against `mergeresolve.ConflictRole`.
  Reed's `Status` lists only the calling session's strands, so another task's conflict session is not reachable.
  Every other parked record and foreign state returns Stuck without a stop or an abort.
  The way-forward clauses name `lyx fabric merge --continue`/`--abort`, `lyx reed remove <guid>`, `lyx loom resume` and `lyx loom goto`, none of which the settings deny.
- **Nil-is-absent seams.**
  `MergeState`, `AbortMerge` and `StopConflictSession` are documented nil-is-absent in `internal/landingshed/deps.go`, skip their step when nil, and `landingDeps` fills all three.
  `TestLandingDeps_EveryFieldPopulated` passes the new argument.
  `envkit.nilLegal` gains the three names.
- **One `MidMerge` answer.**
  `MidMergeState` carries `Verb`, `Source`, `SourceSHA` and `StartSHA` from `loadMergeState`, and landing and the loom verbs read nothing else.
  `refuseOverUnfinishedMerge` reads the parent branch only for a parked `merge-in` at Publish or Finalize, so a read failure never replaces the ordinary refusal.
  `start` and `resume` both pass the current producer.
- **Card 2.**
  `ReadConfigChanges` intersects the fork-point diff with the parent-tip diff, `ParentTip` is set on `ConfigChanges`, and the notice names `parent tip %s`.
  See F2 for one row's strength.
- **Cards 3 and 7.**
  The `Publish: verify failed`, `Publish or Finalize: merge in progress`, `Publish or Finalize: conflict session not stopped` and `decision: task session` refusal-spec rows landed with the code, and the tests reach each reason.
  `decision add` refuses a named strand that differs from the resolved parent, refuses on a resolution error, and lets an empty name through.
  The escalation brief's `lyx loom decision add` instruction is run by a fork of the parent session, so it is not shut out by the new check.
- **Card 4.**
  `conflictRole` has no remaining references, and the `MatchesRole` assertions sit in `TestNumberRole` with no new top-level test.
- **Card 8 and test economy.**
  `runOutcomeStopped` is added to both sub-tables with no production change.
  `TestPreMerge_ClearsOwnParkedMergeIn` and `TestConflictSessionStopper` cover distinct layers, and I found no duplicated test.

### [LOW] The parked record's verb is spelled in three places

**Location:** `internal/loomcli/start.go:143`, `internal/landingshed/premerge.go:14`, `internal/fabricengine/merge.go:231`
**Issue:** `fabricengine` writes `Verb: "merge-in"` as a bare literal, and `landingshed` and `loomcli` each declare their own `mergeInVerb = "merge-in"` to compare against `MidMergeState.Verb`.
A change to the record's verb leaves both comparisons false, which silently disables the Publish/Finalize exception in `start` and `resume` and makes `preMerge.clear` treat the row's own leftover as another merge.
**Fix:** declare the verb once in `fabricengine` next to `MidMergeState`, and have the record writer and both consumers use it.

### [LOW] The "synced into the task" row does not reach the intersection

**Location:** `internal/fabricengine/configchanges_integration_test.go:86`
**Issue:** the row merges the parent's records branch into the task's, and the test then expects `Base` to equal the parent tip.
With `base == parentTip`, `diff base..tip` and `diff parentTip..tip` are the same set, so the row yields no file under the old single-diff code as well.
Only the "task edit identical to the parent's current copy" row reaches the new filter.
**Fix:** drop the row, or build the scenario in which the sync leaves the fork point behind (for example a cherry-picked parent change) so the base stays at the fork and only the parent-tip diff removes the file.
