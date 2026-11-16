# Card 7 — batten-run-asks-act-rule

**Create:**
- `plan:internal/runfsm#Refusal` -> `type Refusal struct`
- `plan:internal/runfsm#Refusal.Error` -> `func (r *Refusal) Error() string`
- `plan:internal/runfsm#Verdict.Refuse` -> `func (v Verdict) Refuse(slug, rerun string) error`

**Edit:**
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#passExitCode`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestPassExitCode`

**Extends:**
- `internal/runfsm/doc.go`

**Uses:**
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#OwnerOf`

**Intent:** `lym batten run` asks the act rule before it acts on a run, and refuses `wait` and `hand_over` naming the way forward (D1).

`Verdict.Refuse` returns nil for `act`, and otherwise a `*Refusal` carrying the verdict, whose text names the slug, the cause and the verdict's way forward, then the rerun.
`startGuard` gains an `Act` seam; `hubStartGuard` fills it with `runfsm.Read` and `runfsm.Act` over the prime,
the owner from `OwnerOf` over the hub's board and `hubgeom.ParentOf`, pause coverage from the hub's pause record,
and the shed-pause aside, which D1's asides give `lym batten run`.
`startGuard.refuse` asks it first for the verb `run`, before every other check, and returns the refusal.
`passExitCode` maps a `*runfsm.Refusal` with a `hand_over` answer to `PassExitOwnedElsewhere` and one caused by pause coverage to `PassExitCovered`,
so the dobby reads a refused pass as it reads those exits today.
The input that reaches each refusal after every upstream check: a run whose batten record carries a cancel intent, or whose effective owner is another machine.

`doc.go` gains a sentence on `Refusal` and the exit mapping it serves.
Tests, tier0: `TestStartGuard_Refuse` gains rows where a `wait` verdict and a `hand_over` verdict refuse `run` with their way forward and leave `step` and `goto` unjudged by it,
and an `act` verdict passes; `TestPassExitCode` gains the two refusal mappings.

**ImpactSummary:** One new seam on batten's start guard, asked for `run` only, and two new rows in the pass exit mapping.

**Commit:** 7: batten-run-asks-act-rule
