```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the round-2 decision record. I traced the code paths that already write, reset or read each state the decisions change.

**Paths traced, per decision**

- D1, `hubgeom.Autonomy` returning a level, against its readers (`loomengine.ApplyAutonomy`, `driftengine.ApplyAutonomy`, pawl `Autonomous`, slackcli `orchAutonomous`, `orchengine/prompt.go`, `orchcli.WithRunAutonomy` `%t` stamp): **survives**. D1 reads both `true` and `unattended` as on and makes the stamp carry the level.
- D2 and D4, the driver rows, against `decideDrive`'s existing budget-exhausted and precondition-unmet hand-backs: **survives** for this lens. Coverage of the table belongs to other lenses.
- D3, counting settle records, against the tackler path in `drive.go` `tackle`, which writes `SettleSpawn` and `SettleRestep` records with `SettlerTackler`: **breaks** (F-a).
- D3, counting over the whole history, against `repairCount` and `wakeRestartGap`, which key on the exact stop key: **survives**. D3 states its count is separate.
- D4, the pause-lift resume, against `drivePasses` and `liftPending`, which give a pass only to drivable runs: **survives as designed**. The record names no verb for the resume (F-f).
- D4, the hold entry, against `syncBoltHold`, which lifts entries of runs held with `runfsm.ReasonHeld` and `BoltHoldCaller`: **survives** only if the hold entry's caller is not `BoltHoldCaller`. D4 gives it its own hold reason.
- D5, the park wait, against batten's `childSpawnError`, `innerrun.go`'s not-parked retry and `innerrun_binaryrestep.go`'s re-step spawn: **survives**, since batten passes the no-wait option.
- D6, the per-segment continue count, against `recordAutoContinue`, which writes `continue` with cause `budget`: **breaks** (F-b).
- D6, its premise about what a `budget` escalation holds, against the Bouncer's other `budget` escalation paths: **breaks** (F-c).
- D6, "continue once", against `circlingVerb`'s hard-cap refusal and `BlockingEscalates` in a window the hard cap opened: **breaks** (F-d).
- D6's bound ("nor the orch"), against `finalizeCirclingVerb`'s accept at Finalize: **breaks** (F-e).
- D6, forcing `BlockingEscalates` under `unattended`, against `shedrecipe/entries_bouncer.go`, which reads `blocking_escalates` from the recipe row only: **survives**, but the mechanism is unstated. That is the handler's call.
- D7, the stamps on the history entry, against shed's fold, which raises `repeats` on the same entry: **survives**, since D7 makes a fold supersede the stamps.
- D11, the `AwaitsReply` split, against its combined-probe callers (loomcli, burlercli and webstercli wiring, `battencli/drive.go`, `gatecli/cardfixer.go`): **survives**, since the held notice keeps the combined probe.

**Findings**

- **F-a**: BLOCKING, design, `_lym/discussion/decision-record.md:131` (D3).
  - D3 says a tackler resume is counted from its tackler attempt "since the tackler's attempts are kept apart from the driver's settle records". That is false: the driver writes a `SettleRestep` record with `SettlerTackler` ("tackler resume") for every tackler resume, and a `SettleSpawn` record per attempt.
  - So under D3's own rules, one tackler-resumed halt counts twice: once from its settle record, which names the halt entry, and once from its attempt. The third-halt verdict is then reached at the second stop, which posts a false severe halt.
  - Evidence: `internal/sheddriver/drive.go:491` and `:533` (the `d.settle(stop, shedrun.SettlerTackler, ...)` calls in `tackle`), `internal/sheddriver/drive.go:913-916`.
- **F-b**: MEDIUM, design, `_lym/discussion/decision-record.md:219` (D6).
  - D6 reads a circling decision file with decision `continue` and cause `budget` as "the segment's continue is spent". The Bouncer's own soft-cap automatic continue writes exactly that file, marked automatic.
  - So any segment where the soft-cap rule continued once reads as already spent, and the arbiter's one continue never happens. Orch and operator continues are indistinguishable too. The record does not say to filter on the automatic flag or on who decided.
  - Evidence: `internal/shedadapters/circling.go:180-181` (`recordAutoContinue` → `createCirclingDecision(..., CirclingContinue, EscalationBudget, true, nil)`), `circling.go:282` (`readCirclingFile` returns `automatic`).
- **F-c**: MEDIUM, design, `_lym/discussion/decision-record.md:211` (D6).
  - D6 says "A `budget` escalation holds an open BLOCKING finding or a red live gate". The Bouncer also escalates with cause `budget` when it cannot read:
    - the ledger;
    - the review;
    - the previous round's decision or ledger;
    - the escalation record;
    - the live state.
  - Under `true`, D6's accept test ("live gate green and every open BLOCKING finding deferred") is vacuously true over an unreadable ledger, so an unreviewed round could be accepted.
  - D6 replaces the arbiter's existing rule "decide nothing at cause `budget` and hand on, naming the input that could not be read" without keeping that rule for the unreadable cases.
  - Evidence: `internal/shedadapters/bouncer.go:1453`, `:1469-1470`, `:1506-1513`, `:1532-1541`, `:1573`, `:1620`; `contracts/stencils/loom/loom-template-arbiter.md:76`.
- **F-d**: MEDIUM, design, `_lym/discussion/decision-record.md:218` (D6).
  - Under `unattended`, D6 forces `BlockingEscalates`. That check runs first in any open soft-cap window, including one opened only by the hard cap, so a round at or past the hard cap with an open BLOCKING finding escalates as `budget`, not `hard-cap`.
  - D6's budget rule then "buys one more round with `lym loom circling continue`", which `circlingVerb` refuses at or past the hard cap. The refusal names accept or raising the cap as the way forward.
  - D6 states the hand-on only for cause `hard-cap`, so the record is silent on the budget-at-hard-cap path.
  - Evidence: `internal/shedadapters/bouncer.go:1449-1454`; `internal/loomcli/circling.go:120-128`.
- **F-e**: MEDIUM, design, `_lym/discussion/decision-record.md:230` (D6 Bound).
  - The Bound says "under `unattended` nothing lands a round with an open BLOCKING finding or a red gate … nor the orch". But `lym loom circling accept` at a Finalize halt whose fix reviews ran out of rounds marks the unapproved fixer change accepted, appends the open findings, and lands it on resume.
  - D6 and D9 give the unattended orch a hold rule only for review rounds, so this path still lands unreviewed findings unattended.
  - Evidence: `internal/loomcli/circling.go:97-98` and `:163-180`; `internal/landingshed/parentalign.go:70-74` (`review-capped`).
- **F-f**: LOW, consistency, `_lym/discussion/decision-record.md:167` (D4).
  - D4 has the dobby resume a run once its pause lifts, but names no verb or path. Today the lift only gives a batten pass to a drivable run.
  - If the resume goes through `lym loom resume`, D5 makes that verb wait up to the keep-warm cap. The record names the no-wait option only for batten, so a dobby tick could block on it.
  - Evidence: `internal/dobbyengine/passes.go:142-163` and `:208-213`; `_lym/discussion/decision-record.md:191-194`.

Tool calls:
1. `syncBoltHold` in `internal/dobbyengine/catchupwait.go`, and `landingshed.HandsBack`. The manifest held neither the bolt-hold lift path nor the landing-halt test.
2. A grep in `internal/landingshed` for the Finalize fix-review halt sentinel and the stop prefixes. Needed to place the Finalize accept path.
3. `internal/landingshed/parentalign.go:60-90`, for the `review-capped` halt values behind `finalizeCirclingVerb`.
