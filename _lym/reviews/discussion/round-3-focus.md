---
round: 3
focus:
  - "D3 halt counting in decision-record.md: check against `tackle` in internal/sheddriver/drive.go, which writes `SettleSpawn` and `SettleRestep` settle records, whether each real stop of a row counts exactly once for every resume path (halt entry, driver settle record, tackler, arbiter settle, re-step after a wake), as the declared PATTERN-stop-settlement entry requires."
  - "D6 `budget` escalations: check `settleUndecided`, `softCapRule`, `blockingEscalationDetail` and `liveRedDetail` in internal/shedadapters/bouncer.go for every cause that escalates `budget`, and settle what the arbiter may accept or continue for each, including an unreadable input."
  - "D6 continue rule: check it against `recordAutoContinue` and `readCirclingFile` in internal/shedadapters/circling.go and the hard-cap refusal of `circlingVerb`; also what bounds the arbiter's `lym board upsert` use to a note."
  - "D6 and D9 unattended accept rule: check every path that can land a change with open BLOCKING findings or a red gate, including `finalizeCirclingVerb` in internal/loomcli/circling.go at the Finalize fix-review halt."
  - "D7, D8 and D10 severe posts: for a `settle: orch` halt, an agent question and a dead driver, check that each escalation reaches a person with the orch's slack button off (`postMessages` in internal/slackengine/outbound.go), is not posted twice, has a defined end for its reminders, and that keyed and keyless dedup never mutes a distinct halt."
  - "D4 and D5 pause and hold: check the hold reason against the one-live-entry-per-scope rule in internal/tenderengine/pause.go, and which verb or path the dobby's lift resume takes (`liftPending` in internal/dobbyengine/passes.go) and whether it can block a tick on the park wait."
  - "D4 stop-row matching: check the rule that no row matches on message text against the bounce-budget row, whose mark is the `shedengine.ReasonBounceBudgetExhausted` prefix read in internal/sheddriver/driverules.go and `arbiterstop.Check`."
---
