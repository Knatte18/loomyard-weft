```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency — whether the changed text still agrees with the decisions, criteria and cross-references around it.

**F1 — MEDIUM — D4's yield bound contradicts the new driver pause-hold exit**
- Changed text: D4, "batten's wiring of the Go driver's `PauseHold` ends the pass as a yield does, exit 0 with the run `running` at its row".
- Sibling left unchanged: the D4 Bound still says "only a producer at a wait that its own next pass re-checks may return it, and InnerRun is the only one".
- Evidence: in `internal/sheddriver/drive.go`, `holdPause` is called at a stop, before a rule action (line 251), before `handOver` (315), before `tackle` (418) and before the arbiter (509). It is never called at InnerRun's wait. So the new pause-hold exit is a second way to end a pass, it is not a producer, and it can happen at any row.
- Twin effect: D4 makes every pass a full pass unless the run sits at Run-Shed. A run held by a pause at Land or Finalize therefore gets a full pass on every probe tick for the whole pause, each one running the origin reconcile and the Open-Window re-run. That is the "Open-Window and origin reconcile on every pass" problem Turn 4 fixed.
- The "Bound: a pass stays in flight only through those waits" sentence also doesn't say what a pass that ends at a stop leaves for the next pass, which re-enters the stop chain.

**F2 — LOW — D10 dropped "or has driven" from the reviewer's proposal**
- Changed text: D10, "from the batten history of every run it drives, read on each tick".
- D4 says dobby runs passes only while batten reads `running`. A hand pass can carry a run through Land and Worktree-Teardown to `done` between two ticks.
- Whether a `done` run still counts as one "it drives" is undefined by D3 and D4. The reviewer's fix said "drives or has driven"; the dropped words leave the landing that F11 was about possibly uncounted.

**F3 — LOW — D6's orch role stencil change has no resume wording**
- Changed text: Goal and D6, "resumes a halted or paused one only by a hand pass (D4)"; D4, "by the orch on the operator's word or after judging the stop".
- D6 lists what the stencil drops (start steps, window wording) but says nothing about the stencil stating when the orch may run the resume pass. The Goal and D4 now assign that duty to the orch.

Tool calls:
- Read `internal/sheddriver/drive.go` at the four `holdPause` call sites. My context showed their line numbers but not the code around them, which I needed to see where the driver's pause hold sits.
