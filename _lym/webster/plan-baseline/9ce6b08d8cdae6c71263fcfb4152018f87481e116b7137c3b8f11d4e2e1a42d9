# Card 3 — The arbiter judges a landing escalate first through the existing seam

**Edit:**
- `internal/sheddriver#ArbiterSeam`
- `internal/sheddriver#goDriver.facts`
- `internal/loomcli#loomArbiterSeam`
- `internal/sheddriver/doc.go`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/loomcli#TestLoomArbiterSeam_JudgesFirst`
- `PATTERN.md`

**Uses:**
- `plan:internal/landingshed#EscalateHalt`
- `plan:internal/landingshed#EscalateStopPrefix`
- `internal/landingshed#LandingStopPrefix`

**Intent:**
D4: `ArbiterSeam.JudgesFirst`, a struct field, becomes `func(producer, reason string) bool`.
`goDriver.facts` passes the stop's `Reason`, or its `Error` when the stop is `FromStatus`, which carries the persisted halt text, as `budgetExhausted` reads both; the rule table and its row order are unchanged.
`loomArbiterSeam`, shared by loom, darn and pawl, marks a blocked stop whose producer is Webster, as now, or one `landingshed.EscalateHalt(producer, reason)` answers true for.
Such a stop goes rule table, arbiter, then parent, with no tackler call, and a blocked stop at Parent-Align or Finalize with any other reason still meets the tackler first.
Batten's arbiter seam sets no `JudgesFirst` and needs no change.

Bound (an edge added): the arbiter can now let a landing go past a decision collision with no orch turn, settling it with its existing repair verb `lym loom decision add` and a `resume` whose goto is the halted row.
Its own rules bound it (it hands on a call needing an operator decision the record does not hold, or one that widens scope), and so does its per-run spawn cap, which stops an escalate-resume loop since an escalate gives its fixer turn back, and the re-run fixer's gate and caps.
The build hand-over row precedes the arbiter-first row, so a new build re-steps the stop once before the arbiter sees it; a Go driver start with no open arbiter attempt runs its start check, which may hand the stop back, and otherwise re-steps the row, so only a re-escalation reaches the arbiter.
A hand-on, an arbiter that is off or out of spawns, and every arbiter fault hand back through the existing hand-on path.

Tests:
- `TestGoDriver_Settle` gains `judgesFirstReason`: when set, the test seam marks a stop only when its producer is `judgesFirst` and the reason it is handed opens with `judgesFirstReason`; the existing `judgesFirst` rows leave it empty and keep their answers.
  A new step builder returns a blocked envelope at Build whose `reason` opens with `escalate: `.
  New rows: that stop with `judgesFirst` Build and `judgesFirstReason` `escalate: ` goes to the arbiter with no tackler call (calls `step`, `arbiter:`), and the arbiter's hand-on hands back; the plain `blocked` stop under the same seam goes to the tackler; a run whose re-check finds the status moved to a blocked Build stop whose `Error` opens with `escalate: ` (via `statusSeq`) sends that status-built stop to the arbiter with no tackler call.
  The restart with an open arbiter attempt and the start check at a halted stop are the existing rows, which act before `JudgesFirst` is read.
- `TestLoomArbiterSeam_JudgesFirst` gains a reason column: Webster with any reason is marked; Finalize and Parent-Align with an `EscalateStopPrefix` reason are marked; Finalize with a `LandingStopPrefix` reason and with a plain reason, Darn-Write with an `EscalateStopPrefix` reason, and the empty producer are not.

Docs: `internal/sheddriver/doc.go`'s arbiter-first rule row says it reads the row and the stop's reason, or a status-built stop's error, and its loom-wiring sentence names Webster and a loom or darn landing escalate.
`PATTERN.md`'s `PATTERN-shed-stop-chain` line becomes, verbatim:

```markdown
- `PATTERN-shed-stop-chain` — A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; a blocked stop the driver's arbiter seam marks as judged first, by its row and reason, goes rule table, arbiter, then parent; arbiter keys in shed.yaml with module overrides.
```

**ImpactSummary:** the `JudgesFirst` field's new signature reaches its one reader in `goDriver.facts` and its one production setter, `loomArbiterSeam`, plus their two tests; batten sets no `JudgesFirst`.
