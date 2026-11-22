```yaml
lens: existing-paths
outcome: reported
```

Lens task: existing-paths. For each piece of state the record introduces or changes, trace the existing code paths that write, reset or read it.

**Paths traced, per decision**

- D1 local-layer refusal (`configengine/layers.go:166` `LocalDenied`, Seed-Child's rewrite of the local layer): survives.
- D1 Finalize's `configChangeNotice` (`landingshed/finalize.go:621`): survives with `autonomy` left out.
- D1 `--layer default` write path (`configcli/doc.go:23`): survives.
- D2 dobby's strict load of `beam.yaml` failing (`dobbyengine/starts.go:95`, `loop.go:237-241`): breaks the rationale's claim that "the orch hears of" it.
- D3 `pr_review_ask_parent` read by `ParentReviewWanted` through `loomengine.LoadConfig` (`loomcli/landingdeps.go:469`): survives if the forcing sits in the load.
- D3 `raddle_auto_approve` read through the run's `setting(module,key)` lookup (`loomcli/raddlewiring.go:135`): breaks.
- D3 raddle mode stored in the run's state on the first call (`loomshed/raddlewrite.go:64-67`): breaks "turning autonomy off restores each key at its next load".
- D3 drift judge keys (`driftengine/config.go`): survive.
- D4 `pr-continue` used as the gate re-try (`landingshed/prreview.go:716` `gateRetryHint`, `loomcli/prcontinue.go:39-43` clearing `MailedHead`): breaks.
- D4 the existing `askParentReview`, which asks once per record and has the orch answer by creel post (`prreview.go:357-392`): breaks, because two answer channels stand side by side.
- D4 `endSession`, `resumeSession` and hold notices (`prsession.go:226-250`, `shuttleengine/doc.go:68`): survive.
- D5 class-5 count against `lym loom decision replace` folding entries (`loom-discussion-spec.md:150-152`): breaks.
- D5 class-5 count against settlements that record no entry, such as `lym loom resume` or `lym batten run` (`orch-role.md:55,163`): breaks.
- D6 the await marker's writers and remover (`pawlengine/landrequest.go:83-127`, `beamawait.go:224,273,300,322`): breaks if the drain marker is kept there; the record names no store.
- D7 `judgeOpenable` (`beamcli/pr.go:240-255`): breaks D7 step 1's premise.
- D7 `mergeParent` filing the merge task released (`beamcli/mergeparent.go:23`): survives.
- D8 `promptHook` setting presence to terminal (`slackcli/hook.go:153`): survives.
- D8 the presence line and `role-slack.md:18` posts to `<shortname>:SLACK`: not governed by the filter, and the record is silent on them.
- D9 `WithPresence` callers (`orchengine/watcher.go:1281,1351,1600`, `orchcli/start.go:836,854`, `resumecontext.go:43`): survive.
- D9 writes to `autonomy` other than `lym config beam`: break the orch's reading.

**Findings**

1. BLOCKING · design · D7 step 1: the step's premise is false.
   - `lym beam pr open` passes a beam that is only behind its parent; it refuses only when a merge task is open or the beam conflicts with its parent (`internal/beamcli/pr.go:240-255`).
   - Running `merge-parent` whenever the beam lags adds a merge-task cycle that is not needed.
   - On a busy parent, each landing lags again, re-drains and runs `merge-parent` again, with no bound.
   - Fix: trigger the step on a conflict, which is the refusal `pr open` names.

2. MEDIUM · design · D4: the `pr-continue` refusal breaks the verb's second use.
   - PR-Review tells the session to run `pr-continue` to re-gate a head whose gate failure lies outside the change (`internal/landingshed/prreview.go:716`); the verb clears `MailedHead` (`internal/loomcli/prcontinue.go:39-43`).
   - Under D4, that re-gate is refused until the parent answers satisfied at the current head. The record is silent on the case.

3. MEDIUM · design · D4: the existing parent-review path runs alongside the new seat.
   - `askParentReview` asks once per record and tells the orch to answer by `lym creel post <strand>` (`internal/landingshed/prreview.go:357-392`).
   - `orch-role.md:199-206` says the same and "the operator ends the review".
   - D4 neither retires nor rewords this, so an orch that answers by creel leaves `pr-continue` refusing with no recorded answer.

4. MEDIUM · design · D3: "forced on the module's resolved config" misses two readers.
   - `raddle_auto_approve` is read through the run's `setting(module,key)` lookup, which a profile's raddle block can set (`internal/loomcli/raddlewiring.go:135-136`, `internal/loomshed/raddlewrite.go:34`).
   - The raddle mode is recorded in the run's state on the first call and taken from there afterwards (`internal/loomshed/raddlewrite.go:64-67`). A change of autonomy mid-run is therefore not applied, against D3's "restores each key's own value at its next load".

5. MEDIUM · design · D6: the drain marker's store is left for the implementer to find.
   - The beam run's existing await marker is rewritten whole by `writeMarker` and `recordDownNotice` (`internal/pawlengine/landrequest.go:83-127`). It is reached from `enter`, `clearDownNotice` and `recheckStanding`, and removed by `landOrRefuse` (`internal/pawlengine/beamawait.go:224,273,300,322`).
   - A drain mark kept there is wiped by one dobby-down notice cycle, and the next wake posts a second `[beam drained]`.
   - Fix: the record names the file and its writers.

6. MEDIUM · design · D5: class 5's count has two holes.
   - `lym loom decision replace` folds the added entries into the main decisions (`loom-discussion-spec.md:150-152`), which resets the count.
   - Settlements that record no entry never count, such as `lym loom resume` on a spent bounce budget (`orch-role.md:163`) or `lym batten run` (`orch-role.md:55`).
   - So "bounds the orch's own settlements at two per row" does not hold.

7. MEDIUM · design · D9: the change mail covers only writes through `lym config beam`.
   - Other paths change the resolved value with no mail:
     - a default-layer change arriving from another machine by hub pull or catch-up (`internal/fabricengine/hubpull.go`);
     - a reconcile (PATTERN-config-layers, Binary-change reconcile);
     - a hand-edited committed file.
   - The orch keeps its stale reading while D2, D3 and D8 read at call time.
   - With D8 in force, an orch that believes autonomy is off writes unmarked replies, and the hook drops every one from Slack.

8. LOW · consistency · D2 Rationale: "a broken file already stops the worktree's dobby, which the orch hears of" holds only after a re-exec.
   - On a first run the dobby only logs the stop (`internal/dobbyengine/loop.go:237-241`).

9. LOW · consistency · D8: the filter governs only the turn-reply hook.
   - The presence line tells the orch to post every operator-meant message to `<shortname>:SLACK` by creel (`internal/orchengine/prompt.go:29`, `contracts/orchrole/role-slack.md:18`).
   - That path bypasses the filter, and the record does not say whether such posts must carry `[operator]`.

**Tool calls**

1. `loomcli/raddlewiring.go` and `landingdeps.go` spans: how the raddle and PR-Review ask keys are read; the manifest did not show the readers.
2. `pawlengine/landrequest.go` marker functions: whether the await marker's writers overwrite the file; not read before.
3. Grep of the shuttle timeout and the worktree layer: whether an autonomous unawaited session times out; inconclusive, and no finding rests on it.
4. Grep of `shuttleengine/spec.go` `Unawaited` and the `configcli` editor paths: the session timeout and the other writers of `autonomy`; not read before.
