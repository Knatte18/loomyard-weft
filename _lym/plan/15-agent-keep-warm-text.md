# Card 15 — agent-keep-warm-text

**Edit:**
- `internal/shellguard#Rows`
- `internal/shellguard#refuseRawRow`
- `internal/shellguard#TestCheckTexts`
- `//contracts/stencils/webster/webster-body-implementer.md`
- `//contracts/stencils/darn/darn-template-write.md`
- `//contracts/stencils/drift/drift-template-verify.md`

**Uses:**
- `//internal/shellguard/doc.go`

**Intent:** D9: the agents' text names the run going on, so no agent reads card-verify's new 75 or the run wait's 75 as a failure; and D7's rule reaches the gate calls the darn and drift stencils spell for their agents.

The gate rule in the raw go row's steer (`Rows`) and in the refuse-raw row's steer (`refuseRawRow`) replaces `When it answers that no gate slot is free yet, run the same command again` with `When lym gate test or lym gate card-verify answers done: false, run the same command again`, keeping the rest of the sentence; the text holds no quote or backtick, which the steer check forbids.
`TestCheckTexts`, tier0, wants `done: false` and `lym gate card-verify` in the refuse-raw steer in place of `no gate slot is free`, and wants the same two in the steer of the raw go row `Rows` returns, so either string going back fails it.
The implementer stencil's slot-busy section says: a `lym gate test` or `lym gate card-verify` call that exits 75 answering `done: false`, with the run's phase and, while the run waits for slots, `slot_free: false` naming the holders, is not a failure: it costs no self-fix attempt, never makes the report `FAILED`, and you run the same command again in a turn that does nothing else, so the prompt cache stays warm between calls; the next call prints the run's result once it ended.
The sentence that a `--wait` call queues for its slot and never exits 75 goes.
`darn-template-write.md` and `drift-template-verify.md` spell their test call `lym gate test` without `--wait`, with the same `done: false` rule in one clause.
Stencil prose carries no test of its own.

**ImpactSummary:** two steer strings, the one test asserting them, and three stencils' prose; no marker or Go-filled value changes.
