```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked every bound, guard and claim about existing code in the record and tried to build a concrete path that breaks each one.

**Claims attacked:**
1. D4: `JudgesFirst` reads the stop's Reason, or a status-built stop's Error. **Held.** Every path that decides a status-built stop carries the halt text in `Error`. Those paths are `recheck` (moved and goto-moved) and `handBackFailure`. Envelope-built stops (step, `resumeOpenAttempt`) carry it in `Reason`.
2. AC: "a driver that **starts** or re-checks at a halted escalate stop, built from the status, also sends it to the arbiter". **Broken for "starts".** See F1.
3. D3: batten's re-step skips the escalate, "so a new build never re-runs the row over an undecided collision; only the arbiter's resume or the orch's `lym loom resume` moves it". **Broken.** See F2.
4. D4 Bound: the driver's build hand-over row comes before the arbiter-first row, so one re-step happens and a re-escalation reaches the arbiter under a new key. **Held.** The successor steps the blocked status and Finalize re-runs. The escalate gave its turn back, so the fixer can run again.
5. D4 Bound: the spawn cap stops an escalate-resume loop, and the loop spends the run's one shared arbiter budget. **Held.** `arbiter_max_spawns` defaults to 3 per run and is counted from attempt records. Each re-escalation gets a new stop key, so each one spawns. A spent budget refuses and hands back.
6. D4: a hand-on, an arbiter that is off or out of spawns, and an arbiter fault all hand back through the existing path. **Held.**
7. D3: `HandsBack` is the one predicate the envelope mark, the driver and `arbiterstop` read. **Held.** The step spec's `HandBack` and the driver's `HandsBack` are both `arbiterstop.HandsBack`, and `Check` reads `landingshed.HandsBack`.
8. D3: the escalate prefix is distinct from `LandingStopPrefix` and `DecisionHaltPrefix`, so neither check matches it, and `arbiterstop.Check` admits it as a blocked stop. **Held** as worded. A new prefix that is itself a prefix of `LandingStopPrefix` would make the escalate predicate match every landing halt. Every reader still puts `HandsBack` first, so nothing observable breaks and I filed no finding.
9. Support log: batten's `HandBack` seam is read by the binary re-step alone. **Held.** Its only non-test reader is the re-step check.
10. D3: a pawl escalate keeps `LandingStopPrefix` and hands back, and `lym loom decision add` refuses a pawl run, which has no decision record. **Held.** The pawl recipe has no Discussion row, and the verb's append refuses a missing record with "the run has no decision record".
11. Constraint: a run an older build halted on an escalate under `LandingStopPrefix` stays a hand-back. **Held.**
12. D1 Bound: a spent cap or over-cap change still reroutes to a review or halts. **Held.**
13. D4: the arbiter settles with `decision add` plus a resume whose goto is the halted row. **Held.** Goto guards such as the Finalize dirt guard can still refuse that resume. The record does not claim otherwise, and a refused resume still ends in the hand-back path.

**Findings:**

- **F1 — LOW, consistency.** `decision-record.md`, Acceptance criteria, the driver bullet.
  - **Summary:** the criterion "a driver that starts … at a halted escalate stop, built from the status, also sends it to the arbiter" names a path that does not exist.
  - **Counterexample:** at start, `goDriver.run` either resumes an open attempt from its saved envelope or runs `checkBlockedStart`. That check builds a status stop only to hand back on a `StartCheck` refusal and decides nothing. Otherwise the driver calls `Step`. Shed re-runs a blocked current producer, so Finalize runs again and the stop the driver decides comes from a fresh envelope.
  - **Effect:** a plan writer is asked to pin a start-from-status arbiter route that the start path never takes. Only the re-check half (`recheck`'s moved stop) is real.
  - **Evidence:** `internal/sheddriver/drive.go:175-205` (run: resumeOpenAttempt, checkBlockedStart, Step); `internal/sheddriver/drive.go:629`; `internal/shedengine/run.go:221-223` ("StateBlocked … deliberately do not short-circuit … re-calls current_producer").

- **F2 — LOW, consistency.** `decision-record.md` D3, the batten re-step sentence.
  - **Summary:** the guarantee "a new build never re-runs the row over an undecided collision; only the arbiter's resume or the orch's `lym loom resume` moves it" is broken by the driver's own build hand-over.
  - **Counterexample:** a deploy lands while the escalate stop waits. `decideDrive`'s hand-over row (`binaryEligible && BuildChanged && !HandedOver`) comes before the arbiter-first row. The driver spawns a successor of the new build, which steps the blocked status. Finalize then re-runs the gate and a fresh fixer session over the undecided collision, with no arbiter resume or orch resume.
  - D4's Bound states this re-step, so D3's absolute wording contradicts D4. It should be scoped to batten's re-step.
  - **Evidence:** `internal/sheddriver/driverules.go:230-235` (hand-over row before the arbiter-first row); `internal/sheddriver/drive.go:175-190` (successor steps at start); `internal/shedengine/run.go:221-223`.

**Tool calls:**
1. grep of `arbitrate` in `internal/sheddriver/drive.go`: to check how an arbiter refusal or fault hands back; the manifest had not read `arbitrate`.
2. Read of `internal/loomcli/finalizeguard.go` lines 1-71: to see whether the Finalize goto guard can refuse the arbiter's resume goto; the manifest had only the `finalizeGuard` signature.
3. grep of the pawl recipe rows and Finalize `mergeParent`'s conflict branch: to confirm pawl has no Discussion row (no decision record) and to see the conflict abort path D1 removes; neither was in the manifest.
