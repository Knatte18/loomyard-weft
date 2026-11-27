```yaml
lens: fix-claims
outcome: reported
```

The fork checked the fix's claims about the code and found three things: two MEDIUM and one LOW.

1. **MEDIUM**, D6, the new sentence listing "the round's review, ledger, previous decision or escalation record" as inputs whose unreadability makes the Bouncer escalate `budget`. This is false for the escalation record. An unreadable escalation record does make `settleUndecided` call `b.escalate(round, EscalationBudget, ...)` (`internal/shedadapters/bouncer.go:1469`). But `escalateWith` reads the record again and returns `shedengine.Stuck` with "bouncer escalation record for round %d is unreadable" (`bouncer.go:1670-1671`), so no `budget` escalation reaches the arbiter. The claim came over from the reviewer's proposed fix.

2. **MEDIUM**, D6, the new first arbiter rule: "An escalation whose record or detail names an input that could not be read, or whose round's review, ledger or live state the arbiter cannot read itself, is handed on at every level". It doesn't cover every unreadable input that D6's own list of causes names.
   - When `softCapRule` cannot read the previous round's decision or ledger, it escalates through `b.escalate(round, EscalationBudget, ptr)` with no detail (`bouncer.go:1532-1541`). `escalate` passes an empty detail (`bouncer.go:1655-1657`), so neither the record nor the detail names the input.
   - The arbiter can read the current review, ledger and live state. The escalation then falls through to the accept or continue rules.
   - So the claim "handed on ... as the arbiter's stencil rules today" fails for those two inputs. The stencil rule is to hand on naming the unread input (`contracts/stencils/loom/loom-template-arbiter.md:76`).
   - The same unreadable review and current ledger from `softCapRule:1505-1513` also carry no detail. The "cannot read itself" clause covers those only if the arbiter happens to read the same file and fail too.

3. **LOW**, D4, the new sentences "The lift gives such a run a batten pass" and "When the dobby lifts a bolt-hold entry that carries a hold reason". The dobby's pass kinds are `passContinue` and `passFull` (`internal/dobbyengine/passes.go:107-112`); no "batten pass" exists by that name. The dobby lifts bolt-hold entries through `syncBoltHold` → `Deps.SyncBoltHold` (`internal/dobbyengine/catchupwait.go:175-187`), which the record doesn't name. The claims are plausible, but they name a path the code doesn't have under that name, so a planner can't locate it.

Every other changed claim matched the code at HEAD:
- The tackler's `spawn` and `restep` settle records and the settler/action constants (`drive.go:491,533`, `shedrun/settle.go`).
- The interrupted-reinvoke re-step is a `go-rule` re-step whose rule names it (`driverules.go:296`).
- The bounce-budget row matches on the reason prefix (`driverules.go:326`, `arbiterstop.go:177`, `shedengine/run.go:28-32`).
- `recordAutoContinue` writes with the automatic flag set, and `readCirclingFile` reads that flag (`circling.go:178-195,282`).
- `circlingVerb` refuses a continue at or past the hard cap (`loomcli/circling.go:120-127`).
- `finalizeCirclingVerb` accepts with the open findings appended.
- `pause.go` adds no entry when the scope already has a live one (`pause.go:42,164`).
- `postMessages` drops the mail of an orch whose slack button is off (`outbound.go:175,200`).

Tool calls:
- `bouncer.go:1436-1490` and `1650-1680`: `settleUndecided` and `escalateWith` were not in context.
- Pass kinds, bolt-hold lift and `arbiterstop` function names: the dobby's pass kinds, how it lifts bolt-holds and the `arbiterstop` symbols were not in context.
- `passes.go:100-116`: the pass-kind constants, to check the "batten pass" name.
