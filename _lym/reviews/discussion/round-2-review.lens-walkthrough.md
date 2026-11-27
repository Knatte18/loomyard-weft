```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end on one concrete case.

**Walkthrough list**

- D1, a board `run_settings` holding a YAML `true`, read through the lookup, `hubgeom.Autonomy` and the notice stamp. Purpose met.
- D2, a precondition-unmet halt at a batten row, run through `decideDrive` and then through D7's stamp. Purpose not met (W5).
- D3, a board claim refused at `Worktree-Create` and resumed by the tackler. Purpose not met: one stop counts twice or more (W1).
- D4, a pause covering a loom child that the driver handed back, then lifted. Purpose partly met (W6).
- D5, an orch `lym loom resume` while the driver holds its driver lock with no wait record. Purpose met.
- D6, case A: round 3 auto-continued by the soft-cap rule, then round 4 escalates `budget` over an open BLOCKING finding. Purpose not met (W2).
- D6, case B: a `budget` escalation from an unreadable ledger, live gate green, under `true`. Purpose not met (W3).
- D6, case C: a `budget` escalation at the perch's hard cap under `unattended`. Purpose partly met (W7).
- D7 with D10, under `unattended`, the orch relays "a run that cannot finish within its record" with its `halt:` key. Purpose not met: two severe posts for one halt (W4).
- D8, a keyed dobby post for halt A, then within the window a keyless orch relay of a scope change (halt B) under `true`. Purpose unclear (W8).
- D9, the orch receives `autonomy: unattended` and an entry overtaken. It holds the run by pause and posts a quiet held line. Purpose met.
- D10, an agent question stall posted as severe under `unattended`, then D8's reminders. Purpose not met: the reminder has no end condition (W9).
- D11, an agent holding its turn with an open question, through `readAgentActivity` and the split probe. Purpose met.
- D12, `unattended` with the slack button off and a D10 stall posted as severe. Purpose not met (W10).
- D13, glossary lines only. Purpose met.

**Findings**

W1 — BLOCKING, design, `_lym/discussion/decision-record.md:128-131` (D3)
- **Summary:** D3's count would count one tackler-resumed halt two or three times, so a class can turn severe at its second stop.
- **Walkthrough:**
  1. The claim refusal leaves a Stuck halt entry with no `routed_to`, which counts once.
  2. The tackler resume appends a settle record with `Settler: tackler`, `Action: restep` (`internal/sheddriver/drive.go:533`). D3 counts "a driver settle record whose rule retried the row: a re-step after a fault".
  3. D3 also counts the tackler attempt.
- D3's premise "the tackler's attempts are kept apart from the driver's settle records" is false.
- D3 dedups only a settle record that names its halt entry. It says nothing to tie the tackler attempt to that entry or to that settle record.
- The arbiter's resumes may also write restep records, `SettlerArbiter` (`internal/shedrun/settle.go:23`). D3 excludes them but names no settler filter.
- **Evidence:** `internal/sheddriver/drive.go:533` (tackle: `d.settle(stop, shedrun.SettlerTackler, shedrun.SettleRestep, "tackler resume", ...)`); `internal/shedrun/settle.go:21-32`.

W2 — BLOCKING, design, `_lym/discussion/decision-record.md:218-221` (D6)
- **Summary:** D6's "continue spent" test matches the Bouncer's own automatic continue, so after any soft-cap auto-continue the arbiter never buys its one round.
- **Walkthrough:**
  1. At round 3, `softCapRule` finds design-wrong MEDIUM entries and calls `recordAutoContinue`. That writes a `continue` decision with cause `budget` (`circling.go:180-181`).
  2. Round 4 escalates `budget` over an open BLOCKING entry.
  3. The arbiter reads `round-3-circling-decision.md`, finds decision `continue` and cause `budget`, and hands on without its continue.
- The file records an `automatic` flag (`readCirclingFile`), but D6's rule does not read it.
- **Evidence:** `internal/shedadapters/circling.go:178-182`, `:280-282`; `internal/shedadapters/bouncer.go:1551`.

W3 — MEDIUM, design, `_lym/discussion/decision-record.md:211-213` (D6)
- **Summary:** D6 states "A `budget` escalation holds an open BLOCKING finding or a red live gate", which is false. Budget also escalates on an unreadable ledger, review or escalation record. Under `true`, "every open BLOCKING finding deferred" is then vacuously true, so the arbiter accepts a round it could not read.
- **Evidence:**
  - `bouncer.go:1453-1454, 1573`: an unreadable ledger escalates `budget`.
  - `bouncer.go:1506-1513`: an unreadable review or ledger in `softCapRule` escalates `budget`.
  - `bouncer.go:1469-1470`: an unreadable escalation record escalates `budget`.
- The arbiter stencil today says "decide nothing at cause `budget` and hand on, naming the input that could not be read" (`loom-template-arbiter.md:76`). D6 does not keep that for an unreadable input.

W4 — BLOCKING, design, `_lym/discussion/decision-record.md:253-257` and `:315-317` (D7 with D10)
- **Summary:** Under `unattended`, the orch relays a halt it classifies with that halt's key. D10's end conditions do not include that relay, so one chase interval later D10 posts the same halt as severe again. D8 mentions again once `slack.remind_after_min` has passed.
- Round 1 F21's fix moved only "no settling action open" to D10's post. "A run that cannot finish within its record" and "a step only the operator can take" are still relayed by the orch.
- The run stays halted under `settle: orch` and the same `halt:` key, waiting for the operator, so D10 chases it and then posts it.
- An agent question the orch relays as severe gets a second post under `question-&lt;id&gt;`, a different key, so D8's dedup does not catch it either.
- **Evidence:** decision-record D7 lines 253-257, D10 lines 308-317, D8 lines 271-274.

W5 — MEDIUM, design, `_lym/discussion/decision-record.md:86-115` (D2/D7)
- **Summary:** The driver hands back several stops today that match no row of D2's table, so D7's `class:` stamp and the halt count have no class to read for them.
- These are: precondition unmet, an interrupted step under the handback policy, an error with no kind or an ownership error, and an awaiting stop with a parent notice or a planned hand-back.
- The "Unnamed stop" row says "the tackler, as today", but the driver hands these back with no tackler (`driverules.go:267-282, 297-298`).
- Whether a third such halt at a row is severe is therefore undefined.
- **Evidence:** `internal/sheddriver/driverules.go:265-302`.

W6 — MEDIUM, design, `_lym/discussion/decision-record.md:165-172` (D4)
- **Summary:** D4 has the dobby "resume" a run after its pause lifts but names no mechanism, and the existing mechanism does not reach the run.
- `liftPending` gives a full pass only to a run whose batten state is running (`passes.go:93-95, 143`). A run that was handed back for its pause is halted at the hand-back.
- If the resume is `lym loom resume`, D5 makes that verb wait up to the keep-warm cap. D5 grants the no-wait option only to batten's calls, so a dobby tick could block on the wait. That contradicts D5's "no dobby or batten tick blocks on the wait".
- **Evidence:** `internal/dobbyengine/passes.go:93-95, 142-163, 210-213`; decision-record D5 lines 191-194.

W7 — LOW, design, `_lym/discussion/decision-record.md:218` (D6)
- **Summary:** Under `unattended`, `BlockingEscalates` fires first in a window the hard cap opened, so a round at the hard cap escalates `budget`, not `hard-cap`.
- D6's budget rule then "buys one more round with `lym loom circling continue`", which `circlingVerb` refuses at or past the hard cap. The refusal names accept or raising the cap as the way forward.
- D6 does not say this case is handed on.
- **Evidence:** `internal/shedadapters/bouncer.go:1449-1455`; `internal/loomcli/circling.go:120-128`.

W8 — LOW, consistency, `_lym/discussion/decision-record.md:273-274` (D8)
- **Summary:** "A post with no key dedups on the slug alone" does not say whether a keyless post is muted by an earlier keyed post of the same slug.
- If it is, a distinct scope-change halt under `true`, relayed without a key within the window after a dobby post for another halt, notifies no phone.
- **Evidence:** decision-record lines 258, 273-274.

W9 — MEDIUM, consistency, `_lym/discussion/decision-record.md:277-278` and `:316` (D8/D10)
- **Summary:** D10 has D8's reminders cover a stall's severe post, but D8's reminders run "until the run leaves the halt".
- A question stall or a dead-driver stall leaves no halt to leave: the run stays `running`. So those reminders either never stop or never start.
- D10's own end conditions (the question's answer or settle, the driver's revive) are not tied to the reminders.
- **Evidence:** decision-record D8 lines 277-278, D10 lines 311-318.

W10 — MEDIUM, design, `_lym/discussion/decision-record.md:259-260` and `:358` (D7/D12)
- **Summary:** With the dobby's orch's slack button off, D7 lets the orch relay in its pane only halts whose notice reads `settle: operator`.
- D10's severe posts, for a `settle: orch` halt, a question or a dead driver, go only to `&lt;shortname&gt;:SLACK`. The relay drops them as handled (`outbound.go:199-200`), and no pane relay covers them.
- Under `unattended` with the button off, those stalls reach no one, not even the pane.
- **Evidence:** `internal/slackengine/outbound.go:199-201`; decision-record D7 lines 259-260, D10 line 316, D12 line 358.

**Tool calls**
- Ran `sed`/`grep` on `internal/sheddriver/drive.go` lines 488-535 and `internal/shedrun/settle.go` constants: to confirm the tackler resume writes a `SettleRestep` record. My context had only the line numbers of the `settle` calls, not their action argument.
