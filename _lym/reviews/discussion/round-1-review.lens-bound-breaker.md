```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens, attacking every bound, guard and claim about existing code in the discussion record.

**Claims attacked**

- D1, the conflict session under the parent lock lengthens the wait, and the lock's owner, pause and release rules bound it unchanged: **held**. `parentlock` has no lease or TTL, a record goes stale only when its owner process dies, and waiters queue on tickets (`internal/parentlock/store.go:238`, `internal/parentlock/doc.go:24`).
- D1, the `finalize` gate and the fixer's caps bound the unreviewed merge: **held** for a fixer that fixes or spends its attempts.
- D1, the retry after a merge-in-required error merges, gates and lands: **held**. The retry runs `mergeAndGate` again under the same lock (`internal/landingshed/finalize.go:319-337`).
- D3, `HandsBack` is the one predicate the envelope mark, the driver and `arbiterstop` read: **held**. Every reader outside `landingshed` goes through `landingshed.HandsBack`, directly or through `arbiterstop.HandsBack` (`arbiterstop.go:63,164`, `loomcli/arm.go:309`, `loomcli/drivewiring.go:302`, `battencli/wire.go:1067`).
- D3, only the escalate changes route, and every other halt keeps `LandingStopPrefix`: **held**. `haltReason` sets the prefix in one return, so the escalate case has to be split out of it (`internal/landingshed/fixerrun.go:297-315`).
- D4, `JudgesFirst` gets "the stop's reason, which the driver's stop already carries": **broken**, finding 1.
- D4's bound that the re-run fixer's gate and caps bound an escalate-resume loop: **broken**, finding 2.
- D4, the per-run arbiter spawn cap stops an escalate-resume loop: **held**. The cap is counted from attempt records on disk, so a restart does not reset it (`internal/shedarbiter/call.go:117,188`). It is the run's whole budget, though, which finding 2 covers.
- D4, a pawl run's arbiter hands an escalate on: **partly held**, finding 3.
- D4, the build hand-over row precedes the arbiter-first row: **held**, by the rule order in `internal/sheddriver/doc.go:36-38`.
- Constraint, an older build's escalate halt under `LandingStopPrefix` stays a hand-back: **held**, because `HandsBack` still matches the old prefix (`internal/landingshed/handback.go:19-21`).
- Auto-mode assumption, the arbiter stencil admits a goto at the halted row: **held**. `loom-template-arbiter.md:66` says "a row at or before the run's current row".
- D2, darn's fallback seam "answers not found": **not attacked**, because the tool budget was spent.

**Findings**

1. **MEDIUM**, class `design`, at D4 and the acceptance criterion "The driver sends such a stop to the arbiter with no tackler call".
   For a stop built from a status, the driver's stop carries the halt text in `Error` and leaves `Reason` empty.
   - Counterexample: a driver restarts at a halted Finalize escalate, or `recheck` rebuilds a moved stop. `stopFromStatus` sets `Reason` only for a paused status and puts the blocked halt text in `Error`. A `JudgesFirst(producer, stop.Reason)` call then reads `""`, and the escalate goes to the tackler first.
   - The record should name the source: the reason, or the error for a stop built from a status, as `budgetExhausted` already reads both. It should also add a criterion for a status-built stop.
   - Evidence: `internal/sheddriver/driverules.go:109-127`, `internal/sheddriver/driverules.go:254`, `internal/sheddriver/drive.go:713-714`.

2. **MEDIUM**, class `design`, at D4's Bound.
   The record says the re-run fixer's gate and caps bound an escalate-resume loop, but an escalate turn spends no fixer attempt.
   - Counterexample: the arbiter resumes with the row as goto, or with a plain re-launch. The fixer escalates again and `Spent--` gives the turn back. The next stop key goes to the arbiter again, and this repeats until `arbiter_max_spawns` is spent.
   - That spawn budget is per run and shared, so the loop can also leave a later Webster stop or tackler escalation with "spawn budget spent", which then hands back.
   - The bound should drop the fixer caps for this loop and say the run's arbiter budget is what it consumes.
   - Evidence: `internal/landingshed/fixerrun.go:256-263`, `internal/shedarbiter/call.go:188-189`, `loom-template-arbiter.md:52,78`.

3. **LOW**, class `design`, at D4, "A pawl run has no decision record to add to, so its arbiter hands an escalate on".
   The claim is stated with no evidence.
   - It holds only on the `decision add` path. The verb refuses a run with no record (`internal/loomcli/decision.go:653-654`), and the stencil says a refusal means hand on (`loom-template-arbiter.md:57`).
   - The arbiter may still choose a plain re-launch or a goto (`loom-template-arbiter.md:52`). The pawl arbiter's recipe description also lists `lym loom decision add` as a repair verb (`internal/loomcli/arbiterprompt.go:22`).
   - The fixer is pointed at a decision-record path on pawl too (`internal/loomcli/landingdeps.go:245`). The record does not say whether that file exists for pawl, or whether a pawl fixer can escalate at all.
   - The record should state the mechanism with this evidence, or limit the claim.

**Tool calls**

1. `grep` of `shedarbiter` for the spawn cap: the cap's scope was not in context.
2. `grep` of `parentlock` for a lease or timeout: needed to attack D1's lock-wait bound.
3. `grep` for `HandsBack` and `LandingStopPrefix` readers outside `landingshed`: needed for D3's one-predicate claim.
4. Arbiter stencil glob: failed to find the file.
5. Arbiter stencil location and its "at or before" goto rule: needed for the auto-mode assumption.
6. Arbiter stencil hand-on and refusal lines: needed for D4's pawl claim and the loop bound.
