# Card 13 — act-rule-hold

**Edit:**
- `internal/runfsm#Inputs`
- `internal/runfsm#Verdict`
- `internal/runfsm#Act`
- `internal/runfsm#Verdict.Refusal`
- `internal/runfsm_test#TestAct`
- `internal/pairteardown#runReader.read`
- `internal/pairteardown#runFakes`
- `internal/pairteardown#TestReadRun`
- `internal/tenderengine#TestResume`
- `internal/pairteardown/doc.go`

**Uses:**
- `plan:internal/boardengine#Owners.HoldOf`
- `internal/runfsm/doc.go`

**Intent:** Make the act rule read a Bolt hold, and name the hold's machine in its refusal (D5).

`ReadRun` fills `Inputs.Held` from the Bolt it reads the owners from: `runReader.read` asks `Owners.HoldOf(slug)` over the `owners` seam card 7 introduced.
`Held` is true while any hold stands, and false for the slug an `adopt-fix` hold names as its fix task, so the fix task acts.
`Inputs` and `Verdict` gain `HoldMachine`, the hold's machine, which `Act` copies into the verdict.
The act rule's order is unchanged: a `landing` run past its parent-side merge already acts under a hold, and everything else waits behind it.

`Refusal` words `ReasonHeld` as `an adopt on machine <m> holds the hub`, with the way forward `wait for that adopt's hold to clear, then run <retry> again; if machine <m> has stopped, ask the operator`.
It names neither `lym tender resume` nor `--take-hold`.
An empty `HoldMachine`, as a hand-built input gives, words the machine as `another machine`.

Tests, tier0:
`TestAct` gains rows: the held refusal names the machine and no `--take-hold`; an empty `HoldMachine` words `another machine`; a held run at `State: Landing` with `PastParentMerge: false` waits (the Finalize cut between its local merge and its parent push leaves origin's parent branch alone under a hold), and the same inputs with `Held` false act, so its parent push follows at the first pass after the hold clears.
`TestReadRun` gains rows: no hold reads not held; an `adopt` hold reads held with its machine; an `adopt-fix` hold reads its fix task not held and another slug held; `runFakes`' owners carry the holds.
`TestResume` in `internal/tenderengine` asserts the reworded held refusal in place of `an adopt holds the hub`.
`pairteardown/doc.go` names the hold read.

**ImpactSummary:** Two struct fields and one refusal wording in runfsm; ReadRun's one reader fills Held, which no production code set before, so every act-rule caller now waits under a hold.
