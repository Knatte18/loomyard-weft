# Card 10 — bouncer-harden-seams

**Create:**
- `plan:internal/shedadapters#TestWriteRoundFacts_HardenSection` -> `func TestWriteRoundFacts_HardenSection(t *testing.T)`
- `plan:internal/shedadapters#TestBouncer_HaltRunsTheHaltSeam` -> `func TestBouncer_HaltRunsTheHaltSeam(t *testing.T)`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#roundFacts`
- `internal/shedadapters#writeRoundFacts`
- `internal/shedadapters#renderRoundFacts`
- `internal/shedadapters#Bouncer.settleUndecided`
- `internal/shedadapters#Bouncer.softCapRule`
- `internal/shedadapters#Bouncer.escalateWith`
- `internal/shedadapters#TestBouncer_AutoAccept`
- `internal/shedadapters#TestBouncer_SoftCapRule`

**Extends:**
- `internal/shedadapters/doc.go`

**Intent:**
The Bouncer seams a Harden-Bouncer needs (D5, D6).

`BouncerConfig` gains three optional fields, nil on every existing Bouncer:
- `RoundFacts func(round int) (string, error)`: a section the facts file renders after the reviews and ledgers. `roundFacts` carries it and `renderRoundFacts` writes it under its own heading; `writeRoundFacts` calls it, and a call error fails the judge prompt with the error, as a ledger read error does.
- `LiveRed func(round int) (bool, error)`: when it answers true for the round, `settleUndecided` and `softCapRule` never turn a spent budget into an accept. The round escalates instead through `escalateWith` with the budget cause and a detail naming the red live report, as a capped campaign does, so no campaign lands past a red live gate without a parent decision. An error from it escalates the same way, its text in the detail.
- `OnHalt func() error`: `escalateWith` calls it before it writes a new escalation record whose cause is circling, budget or hard-cap, the causes that halt a campaign capped or circling, so the campaign reaps its sandbox as it halts. A decision or stalemate escalation, and a re-step that returns an escalation already recorded, never call it. An error is logged at Warn and the escalation goes on, its way forward naming `lym harden sandbox clear` in the task worktree.

`TestBouncer_AutoAccept` gains rows: with auto-accept set and the budget spent, a red `LiveRed` escalates and never accepts, a `LiveRed` error escalates with its text in the detail, and a green one accepts as before.
`TestBouncer_SoftCapRule` gains the same red row for the soft-cap auto-accept.
`TestWriteRoundFacts_HardenSection` is tier0: a set `RoundFacts` renders its section, a nil one renders none, and its error fails the call.
`TestBouncer_HaltRunsTheHaltSeam` is tier0: a circling escalation and a hard-cap escalation each call `OnHalt` once before the record is written; a decision escalation, a stalemate escalation and a re-step over a recorded escalation never call it; a failing `OnHalt` still escalates.
Card 11 fills the three fields from a Harden-Bouncer row's config.

**ImpactSummary:** Three optional Bouncer config fields with nil defaults; every existing Bouncer row leaves them nil and behaves as before.
