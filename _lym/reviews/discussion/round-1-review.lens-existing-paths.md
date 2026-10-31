```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. For each piece of state the decisions add or change, I traced the code that already runs and checked whether the decision's result survives it.

**Paths traced (per decision):**
- D7, watchdog `agentActivityFor`: an unawaited session's activity counts only while the run record's PID is alive and its turn end is not held. PR-Review and Beam-Await write no trace lines on a normal wake, and `reviewWaitingFor` and `markWait` are display only. **Breaks Open risks bullet 2:** the code already answers that question (F1).
- D7, dobby `childAtWaitingRow`: it reads only a `running` status, so a halted Sampler-Session still gets the quiet notice as before. Survives.
- D7, reed watchdog daemon: it has no reap rule keyed on a gone waiter, so a reused session survives. Survives.
- D3, Finalize `reviewFollowsReroute` and `keepFixerCommits(..., false)`: a halt for the orch once the reading follows the gate. Survives.
- D3, `CodeReviewOff`: only Parent-Align reads it, and sampler has no Parent-Align row. **Misread premise** (F3).
- D3, entry gate `enter` on a reroute into a gated row: the chain-skip routes by `on_done`, and Finalize halts first when the perch is off. Survives.
- D2/D3, Seed-Child writing a profile block into the local layer: `sampler.*` keys are open to the local layer unless declared guard keys. Survives.
- D6 stamp, `appendOrFold`: Done entries never fold, so the count is exact. Survives.
- D6 stamp, `lym shed goto`: it appends an `OutcomeGoto` entry, not Done, so a goto back without an intervening Done keeps the stamp. **Breaks** the D6 acceptance criterion (F2).
- D6 stamp, Finalize's `on_reroute: Code-Bouncer`: the reroute never re-enters Sampler-Session. **Misread premise** (F2).
- D6 done record, `removeRunRecords` (cancel and cleanup): it removes the status history and empties scratch together, so stamp and flag reset together. Survives.
- D4, `decisionVerb` caller checks plus the creel notice that skips the caller's own run: the widened rule sits beside them. Survives.
- D5, `ensureSession`: it reuses a live strand without re-recording the run PID. Survives for D5, but this is what F1 rests on.

**Findings:**

F1. MEDIUM, design. Location: Open risks bullet 2, Constraints watchdog fact, D7.
Summary: the record says nobody checked whether PR-Review and Beam-Await are already exempt from the step watchdog, but the code answers it now: they are not exempt.
- Evidence (activity reading): `agentActivityFor` counts a wait marker only when it is not held, and an agent run only while its run is live (`internal/loomcli/looparm.go:155-185`). `runIsLive` requires the recorded PID to be alive (`internal/shuttleengine/activity.go:125-126`). That PID is the starting step's own (`internal/shuttleengine/run.go:689`).
- Evidence (no other activity source): PR-Review's `wake` writes no trace line on a normal wake (`internal/landingshed/prreview.go:406-470`), and Beam-Await logs only Warns. `reviewWaitingFor` only feeds the status display (`internal/shedverbs/spec.go:95-103`).
- Evidence (arming): loom arms the watchdog for every loom recipe, pawl included (`internal/loomcli/arm.go:431`).
- Consequence: today both rows are killed after `step_idle_timeout_min` of quiet. This is a deferred research question the code settles.
- Fix: record this as a Constraints fact and drop Open risks bullet 2.

F2. MEDIUM, design. Location: D6 entry-stamp bullet and the D6 acceptance criterion "a done flag written before a reroute or `goto` back to the row is ignored after it".
- `lym shed goto` appends an `OutcomeGoto` entry, not Done (`internal/shedengine/goto.go:135`), so the Done count does not change.
- Counterexample: the run is halted at Sampler-Session with a flag written (D6 accepts that). The operator then runs `goto Record-Write`, and the run comes back to Sampler-Session. The stamp still matches, so the row ends Done at once on a clean checkout, without the operator saying the work is done.
- Finalize's reroute targets Code-Bouncer (`contracts/recipes/darn-recipe.yaml`, Finalize `on_reroute`; D1 keeps it), so "by Finalize's reroute" names a return to the row that cannot happen.
- Fix: say how a goto that re-enters without a Done treats the flag, or clear the flag whenever the row is left. Drop the reroute case.

F3. LOW, consistency. Location: D3, the sentence naming "the landing's `CodeReviewOff` in `internal/loomcli/landingdeps.go`".
- `CodeReviewOff` is read only by Parent-Align (`internal/landingshed/parentalign.go:435`, `internal/landingshed/deps.go:320-322`), and sampler has no Parent-Align row (D1).
- Naming it as a perch-off reading on a sampler run is a wrong premise. Only `rerouteReviewed` and `codeBudgetZero` reach a sampler Finalize (`internal/loomcli/landingdeps.go:80-83, 390-409`).

Tool calls:
1. `internal/shedrun/paths.go` `ScratchDir`, plus a grep for who removes scratch: the manifest did not cover where the done record lives or what clears it.
2. `internal/shedengine/goto.go` history entry, plus local-layer key rules in `configreg`/`darnengine`: the manifest lacked goto's history write and whether a seeded `sampler.*` key is local-open.
3. `internal/battencli/cleanup.go` `removeRunRecords`: to check that the status history and the scratch done record are reset together.
4. `removeRunRecords` callers (`cancelcmd.go`, `cleanup.go`): to confirm the reset paths are cancel and cleanup only.
5. Reed reap criteria grep: to check whether a reused session whose recorded waiter PID is gone could be reaped.
