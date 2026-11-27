```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked the claims, bounds and guarantees the decision record states.

**Claims attacked**

1. D6, "this per-segment count bounds the continue, read from the circling decision files": broken (BB1).
2. D6, "A `budget` escalation holds an open BLOCKING finding or a red live gate": broken (BB2).
3. D6, "Otherwise it buys one more round with `lym loom circling continue`": broken at the hard cap (BB3).
4. D6 Bound, "under `unattended` nothing lands a round with an open BLOCKING finding … neither the Bouncer, nor the arbiter at any cause, nor the orch": broken by Finalize's fix-review accept (BB4).
5. D3, "a tackler resume, counted from its tackler attempt, since the tackler's attempts are kept apart from the driver's settle records": broken (BB5).
6. D3, "one that names a halt entry counts with that entry as one stop": held for go-rule restep records only; it does not cover tackler records (folded into BB5).
7. D4 Bound, "never a run the orch holds", and the hold entry's own reason: broken (BB6).
8. D5, "so no dobby or batten tick blocks on the wait": broken by D4's dobby resume (BB7).
9. D7 and D10, "so one halt gets one severe post": broken for the other classes the orch relays (BB8).
10. D8, "a post with no key dedups on the slug alone … a new halt of the same run … notifies again": broken for a keyless halt (BB9).
11. D6, "`lym board upsert` with kind note, which the arbiter's settings allow for that kind alone": unpinned (BB10).
12. D4, "The budget's hand-back already marks it … so no rule matches on message text": contradicted by the code (BB11).
13. D1, "A value the lookup refuses reads as `false` there": held (`hubgeom/autonomy.go` reads any failure as false).
14. D5, "the verb answers at once for a driver that keeps a wait record": held for a halted run. On a running run, `resume.go:118-121` already answers "already running".
15. D7, "a restarted dobby posts it once more, which is accepted": held; the record states that bound itself.
16. D12, "warns once per episode": held.

**Findings**

- **BB1 — BLOCKING, design, `_lym/discussion/decision-record.md:218-221` (D6)**
  - Summary: the arbiter reads a `continue` file with cause `budget` as its spent continue, but the Bouncer's own automatic soft-cap continue writes exactly that file. So does every orch continue under `true`.
  - Effect: in a segment where the soft-cap rule auto-continued once, the arbiter never continues. In a segment with none, nothing in the file tells the arbiter's continue apart from the orch's. The bound counts something other than the arbiter's continues.
  - Counterexample: the soft-cap rule auto-continues round 3. Round 4 escalates `budget`. The arbiter finds `round-3-circling-decision.md` with decision `continue` and cause `budget`, so it hands on at once.
  - Evidence: `internal/shedadapters/circling.go:180-181`, where `recordAutoContinue` writes `CirclingContinue, EscalationBudget, true`. `internal/shedadapters/circling.go:282`: `readCirclingFile` returns an `automatic` flag that D6 does not read.
- **BB2 — BLOCKING, design, `_lym/discussion/decision-record.md:211-213` (D6)**
  - Summary: D6's premise that a `budget` escalation holds an open BLOCKING finding or a red gate is false. The Bouncer also escalates `budget` when it cannot read the round's review, its ledger, the previous decision, the escalation record or the live state.
  - Effect: with an unreadable ledger and a green gate, "every open BLOCKING finding is `deferred`" holds vacuously. Under `true` the arbiter then accepts a round whose findings no one could read.
  - Evidence: `internal/shedadapters/bouncer.go:1506-1514` (`softCapRule` on an unreadable review or ledger), `bouncer.go:1534`, `bouncer.go:1469-1470`, and `bouncer.go:1573` (`blockingEscalationDetail` on an unreadable ledger).
- **BB3 — MEDIUM, design, `_lym/discussion/decision-record.md:218` (D6)**
  - Summary: D6 hands on only cause `hard-cap` at the hard cap. A `budget` escalation can sit at or past the hard cap too, and there `lym loom circling continue` refuses. D6's budget rule never says what the arbiter does when its one continue is refused.
  - How it arises: the soft-cap window opens at the hard cap, and `BlockingEscalates` runs first in that window, so the round escalates with cause `budget`, not `hard-cap`. Under `unattended`, D6 forces `BlockingEscalates` on every perch, so this case comes up there.
  - The refusal's way forward names accept, which D6 forbids under `unattended`.
  - Evidence: `internal/shedadapters/bouncer.go:1449-1455` and `internal/loomcli/circling.go:120-127`.
- **BB4 — MEDIUM, design, `_lym/discussion/decision-record.md:228-230` (D6 Bound)**
  - Summary: a run halted at Finalize after its fix reviews ran out of rounds is settled by `lym loom circling accept`. That accept lands the fixer's change with its open findings appended.
  - D6 constrains Bouncer rounds and escalations only. Its "nothing lands … nor the orch" bound leaves this path to the orch under `unattended`, and to the arbiter if that halt's reason is not a `LandingStopPrefix` hand-back.
  - Evidence: `internal/loomcli/circling.go:158-180` (`finalizeCirclingVerb`), `internal/arbiterstop/arbiterstop.go:99-106` (a blocked halt is eligible unless it is a landing or decision halt) and `internal/landingshed/handback.go:13-18`.
- **BB5 — BLOCKING, design, `_lym/discussion/decision-record.md:128-131` (D3)**
  - Summary: the premise that the tackler's attempts are kept apart from the driver's settle records is false. A tackler resume writes a `SettleRestep` settle record with settler tackler, beside its spawn record and its attempt directory.
  - Effect: D3 counts that resume twice, once as a settle record "whose rule retried the row" and once from its tackler attempt. If the stop also left a Stuck halt entry, the tackler attempt adds a third count, since D3 merges only a settle record that names its halt entry.
  - Counterexample: a board-claim refusal the tackler resumes twice reaches the third halt at the row at its second stop, so `settle: operator` comes one stop early.
  - Evidence: `internal/sheddriver/drive.go:491` (`SettlerTackler, SettleSpawn`), `drive.go:533` (`SettlerTackler, SettleRestep, "tackler resume"`) and `internal/sheddriver/driverules.go:237-244` (`repairCount` counts every `SettleRestep` whoever the settler is).
- **BB6 — MEDIUM, design, `_lym/discussion/decision-record.md:170-172,177` (D4)**
  - Summary: the pause record keeps one live entry per scope, and a pause of a scope that already has a live entry adds nothing. The orch's hold can therefore fail to write its hold reason.
  - Counterexample 1: the slug already holds a live slug-scoped pause, such as the operator's own. The orch's hold pause adds no entry and no hold reason. When the existing entry is lifted, D4's lift resumes a run the orch holds.
  - Counterexample 2: an adopt's bolt-hold entry holds the slug. The dobby lifts that entry itself once the adopt's hold clears. D4's lift then resumes the run, and the hold stood only as long as the bolt-hold did.
  - Evidence: `internal/tenderengine/pause.go:42` ("A scope that already had a live entry adds none"), `pause.go:144`, `internal/tenderengine/bolthold.go:117` and `internal/pauserecord/record.go:68` ("at most one live entry per scope").
- **BB7 — MEDIUM, design, `_lym/discussion/decision-record.md:167,191-194` (D4 against D5)**
  - Summary: D4 has the dobby resume a paused run once its pause lifts, but names neither the verb nor the no-wait option. D5 makes `lym loom resume` and `lym loom start` wait up to the keep-warm cap, and gives the no-wait option only to batten's child start and resume.
  - Effect: if the dobby's lift resume goes through `lym loom resume`, its tick blocks on the park wait. That breaks D5's claim that no dobby tick blocks on the wait.
  - Evidence: `internal/dobbyengine/passes.go:181-233`; `drivePasses` runs on the dobby's tick.
- **BB8 — MEDIUM, consistency, `_lym/discussion/decision-record.md:253-257,316-317` (D7 and D10; carries F21 in part)**
  - Summary: under `unattended` the orch still relays two classes itself while the run stays halted under its `settle: orch` entry: "a run that cannot finish within its record" and "a step only the operator can take". D10's chase ends only on a transition or a pause, never on the orch's relay, so one interval after the chase the dobby posts the same halt again with the same key.
  - Effect: the "one severe post" claim holds only for the no-settling-action class. The second post is a second mention once it falls outside `slack.remind_after_min`.
  - A question the orch relays as "cannot finish within its record" is chased under `question-&lt;id&gt;` instead, which is a distinct key and notifies again.
  - Evidence: D10's own end conditions in the record, and `internal/slackengine/outbound.go:179-215`, which has no dedup of its own.
- **BB9 — LOW, design, `_lym/discussion/decision-record.md:273-274` (D8)**
  - Summary: "a post with no key dedups on the slug alone" mutes a keyless relay that follows any severe post of the same slug inside `slack.remind_after_min`.
  - Counterexample: under `true`, a scope change the orch relays without a key, within the window after a lym-posted severe halt of the same run, notifies no phone. That contradicts "a new halt of the same run … notifies again".
  - Evidence: D8's text, and `internal/slackengine/outbound.go:2-3`, where the relay's only per-process memory is the outbox.
- **BB10 — LOW, design, `_lym/discussion/decision-record.md:214-215` (D6)**
  - Summary: "which the arbiter's settings allow for that kind alone" rests on the permission settings matching inside the JSON argument of `lym board upsert`. The record pins neither that tool behaviour nor a board-side refusal.
  - Effect: as written, the grant lets the arbiter upsert any entry's body, not only file a note. The board refuses task creation, but not edits of other notes or entries.
  - Evidence: D6's text; I found no settings test cited for it.
- **BB11 — LOW, consistency, `_lym/discussion/decision-record.md:160-161` (D4)**
  - Summary: "The budget's hand-back already marks it, as the arbiter's batten check reads it" stands beside "so no rule matches on message text". The existing mark is a prefix of the error text.
  - Evidence: `internal/arbiterstop/arbiterstop.go:177` and `internal/sheddriver/driverules.go:322-327` both use `strings.HasPrefix(..., ReasonBounceBudgetExhausted)`.

**Tool calls**

1. Grep of the `d.settle(` calls in `drive.go` and the matching in `syncBoltHold`: my context lacked the tackler's settle-record writes and how a bolt-hold entry is lifted.
2. Grep of `landingshed.HandsBack` and the pause per-scope rule in `tenderengine`: my context lacked the Finalize hand-back scope and whether a second live pause entry per scope is refused.
