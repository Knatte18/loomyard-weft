```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough lens: one concrete case run step by step through each decision's mechanism, with the end state checked against the decision's purpose.**

**Findings**

1. **BLOCKING — D8 with D6: Finalize's halts point at a verb that refuses at Finalize.**
   - Case: the beam has no record, and beam.yaml requires a pull request (a rule set after ready). Finalize halts naming `lym beam pr open <slug>` and `lym loom resume`.
   - The orch runs `lym beam pr open`. D6 refuses every `pr` verb whose pawl run is not at Beam-Await, and the run is at Finalize. D10 removes any way back to Beam-Await.
   - The same dead end follows "a push the parent refuses as protected halts naming `lym beam pr open`". By then the parent-side merge has also already run: `finalize.go` step 6 calls `markTaskDone` and `recordLanding` before `pushParent`.
   - Either D6 lets `open` run while the run is halted at Finalize, or these halts name another way forward.

2. **MEDIUM — D8: "squash-merges the recorded pull request" is not carried by any value the mechanism reads.**
   - `mergeOnGitHub` squashes on `Config.Squash || landing.atOnce`, and `atOnce` is set only by `MergeAtOnce`, which only beam.yaml `merge` sets.
   - Case: a record stands and the parent's beam.yaml is `on`. The resolver's yes has `MergeAtOnce` false, so a hub with `squash: false` gets a merge commit on GitHub and a non-squash parent-side merge.
   - D8 needs to say the yes from a beam record marks the landing to squash, the way `atOnce` does.

3. **MEDIUM — D13 with `PATTERN-shed-stop-chain`: the "halted" move may not stay paused.**
   - Case: a run at PR-Review is stepped by the new binary. Shed's step moves it to Beam-Await and halts it.
   - If the driver sees that halt as a stop, the chain runs (rule table, tackler, arbiter, parent), and the arbiter may resume the run before the orch acts.
   - The record does not say whether the move's halt bypasses the chain or is judged as a parent-only stop. The entry asks that the run stay "paused, and the beam's orch is told".

4. **LOW — D13: the dobby-stop marker is left standing.**
   - Case: a run at Parent-Align. Beam-Await's accept had written the dobby-stop marker and stopped the dobby. D13 removes the land request and the ready record but not the marker.
   - On resume, Beam-Await re-enters with the await marker standing and posts the dobby-down notice. A later `refuse` reads the stale marker and restarts the dobby.
   - Harmless, but the move should remove the marker or say why it stays.

5. **LOW — D11: a long Beam-Await can run out of re-invokes.**
   - Re-invoke runs through `sheddriver/driverules.go` only while `facts.RepairCount < driveRepairCap`.
   - Beam-Await can wait for days, so after the cap's number of loop exits the fix for #57 stops working. The record should say whether the repair count resets or how it is bounded.

6. **LOW — D3, D5 and D6: "at Beam-Await" does not say whether a halted run counts.**
   - Today `judgeLandRun` requires a running run.
   - After D13, a moved run is halted at Beam-Await. If the orch runs `lym beam describe` or `lym beam pr update` before `lym loom resume`, it is refused or accepted depending on that reading.
   - State running-only (the D13 notice already orders resume first) or allow halted.

**Decisions walked**
- D1: rows rewired, Beam-Await to Finalize. Met.
- D2: ready, then the dobby's land request, then accept writes Finalize, then the re-check, then Done; a member filed in the window is refused by the re-check. Met.
- D3: ready with open work refuses through BeamLandRefusal; parent ahead refuses naming merge-parent. Met.
- D4: `pr open` writes the record, and Finalize reads the same path. Met, given D8's resolver input.
- D5: describe writes `summary.md` in `_lym/landing`, the path Finalize reads. Met.
- D6: open, update, status and close at Beam-Await. Met at Beam-Await; broken from Finalize (finding 1).
- D7: a member landing moves the beam tip, the dobby announces past the last head, and `describe` commits to weft, which adds no first-parent commit. Met.
- D8: record standing, mergeAndGate, then mergeOnGitHub, then the parent fast-forward, then the parent-side merge on the already-landed path. Order met; squash not met (finding 2); the halt ways forward don't work (finding 1).
- D9: no record and `merge`, so openAtOnce finds an open PR and leaves its text unedited. Met.
- D10: reopen is removed and PR-Review drops its reroute. Met.
- D11: loop exits at Beam-Await, the policy is reinvoke, and sheddriver re-steps. Met within the cap (finding 5).
- D12: start on a beam held at Finalize brings up the orch only. Met.
- D13: PR-Review status moves to a halted Beam-Await. Pause not assured (finding 3); marker left standing (finding 4).
- D14: role text only. Nothing to run.

**Tool calls**
1. grep for the readers of `InterruptPolicyFor` and `loopExitedEarlyStop`: to see who re-steps an interrupted row, which my context lacked.
2. grep of the Finalize step names in `finalize.go`: to find the order of pullRequestLanding, merge and gate.
3. Read `finalize.go` lines 255–365: the Finalize step order D8 relies on, which my context lacked.
4. `loopwait.go` deadLoopStop and the uses of `InterruptPolicyReinvoke`: to find the re-invoke consumer and its repair cap.
