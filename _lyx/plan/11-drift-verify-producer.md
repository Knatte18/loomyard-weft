# Card 11 — drift-verify-producer

**Create:**
- `plan:internal/driftengine#VerifyTold` -> `type VerifyTold struct`
- `plan:internal/driftengine#DriftVerify` -> `type DriftVerify struct`
- `plan:internal/driftengine#NewDriftVerify` -> `func NewDriftVerify(name string, row Row, verify shuttleengine.GateEntry, deps Deps) *DriftVerify`
- `plan:internal/driftengine#DriftVerify.Call` -> `func (p *DriftVerify) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/driftengine#TestDriftVerify` -> `func TestDriftVerify(t *testing.T)`
- `plan:internal/driftengine#gateLoopSeats` -> `func gateLoopSeats(t *testing.T, actions []func()) *shedfake.SeatRunner`

**Edit:**
- `plan:internal/driftengine#Deps`
- `internal/shedadapters#DecisionHaltPrefix`

**Extends:**
- `internal/driftengine/doc.go`
- `pattern/PATTERN-gate-slot.md`
- `internal/shedadapters/doc.go`

**Uses:**
- `plan:internal/driftengine#VerifyTable`
- `plan:internal/driftengine#RunCompare`
- `plan:internal/driftengine#CountedCapture`
- `plan:internal/driftengine#ResetRuns`
- `plan:internal/driftengine#AttemptHead`
- `plan:internal/driftengine#ScopeRecord.Unruled`
- `internal/shedadapters#NewMultiLLMProducerSourced`
- `internal/loomshed#NewVerifyGate`
- `internal/testkit/shedfake#SeatRunner`

**Intent:**
Add `Drift-Verify`: the mechanical compare, then one strong agent on drift, and the decision halts that put drift in a task meant to change nothing to the parent (D6, D7).

`Deps` gains `VerifyTable func(told VerifyTold) (seatengine.Table, error)`, `Compare func(ctx context.Context) (CompareResult, error)` (built by the wiring over `RunCompare` with the scope's `SkipList`) and `GateFindingsFile string`.
`VerifyTold` carries the failing compare's log and the baseline's commit (the counted capture's `head`), which the table renders.
`verify` is the row's resolved `verify` gate entry, the `drift-verify` key's call through `loomshed.NewVerifyGate` and `verifytree.Verify`, built by the registry entry (card 14-drift-recipe-rows).

`DriftVerify.Call` commits `_lyx/drift` (report, scope record, logs, any capture log still uncommitted) through `Commit` on every way out, and in order:
1. no scope record, or scope `off`, is Done naming that reason; a waived scope is Done without a compare.
2. Under `full`, when the scope's `halt_named_at` is set and `Status`'s last history entry is a Stuck of this row whose latest time (`LastAt` when set, `At` otherwise) is at or after it, this entry is the resume from that halt: every named test still on `accepted` is marked `ruled` and `halt_named` is cleared, before any compare.
3. Under `full`, any `unruled` acceptance ends the row in a decision halt before any compare (below).
   So an agent session that accepts and ends in an error, transient or interrupted stop leaves an acceptance no halt recorded, and the resumed entry, a tackler's included, halts for the parent instead of comparing.
4. No counted capture (`CountedCapture` with `AwaitRow`) is Stuck with a reason opening with `shedadapters.DecisionHaltPrefix` under every scope, naming `lyx drift waive --reason <text>` followed by `lyx loom resume`.
5. `Config().Step("compare")` refusing is Stuck naming `lyx config drift` then `lyx loom resume`.
6. The handoff's first two checks again, so a resume from a dirty-tree or test-gate halt never compares past what it guarded (D6's order on every way out): a dirty tracked tree is Stuck naming the paths and committing or cleaning them, then `lyx loom resume`; when HEAD differs from `AttemptHead`, the told `verify` entry's gate runs once, and a failure is Stuck naming its findings file and fixing the tests, committing, then `lyx loom resume` (a tree its record already passed skips without running).
7. The mechanical compare, which counts no run: a pass is Done and spawns nothing.
8. A failing compare spawns one agent through `shedadapters.NewMultiLLMProducerSourced` over `Seats` and `VerifyTable`, whose fresh-spawn hook calls `ResetRuns` with HEAD, so each fresh attempt has `verify_max_runs` runs; each resume reaching a failing compare spawns a fresh agent, bounded by the run's tackler and arbiter spawn budgets (D6).
   The table's gate is, in order: `dirty` (3 attempts, failing with the dirty paths and asking the session to commit them); the told `verify` entry, wrapped to pass without running while HEAD equals `AttemptHead`, so it runs only once the session committed code; and `compare` (one attempt, must-pass), which runs `Compare` and on a failure returns a `Terminal` result naming the log, so no fix the agent or a gate re-prompt makes goes uncompared.
9. The agent's end: an error is returned as the seat producer maps it (an error or transient stop, never settled with an acceptance); a Done is re-checked as in step 3 under `full`, else Done; a Stuck after the compare's terminal failure names the report and the ways forward, `lyx drift accept <test> --reason <text>` followed by `lyx loom resume`, or fix and `lyx loom resume`; a Stuck after the dirty or verify entry spent its attempts keeps the gate's findings file and names its way forward, committing or cleaning the tree, or fixing the tests and committing, then `lyx loom resume`, whose entry re-runs step 6.
The producer tells the compare's terminal failure from an exhausted entry by the chair's `GateOutcome.Entries`, the entry in state `failed`.

Every `Drift-Verify` Stuck under `full`, and under `partial` the no-counted-capture halt and a final-compare failure while `withdrawn` is non-empty, has its reason opened with `DecisionHaltPrefix`, so `arbiterstop.HandsBack` hands it to the parent, ahead of the tackler and the arbiter (D6).
Every decision halt under `full` reached while an `unruled` acceptance stands names those tests and the parent's ways forward for them beside its own: `lyx loom resume` to stand by them, after any `lyx drift accept` of the parent's own, or `lyx drift withdraw <test>` and `lyx loom resume` to reject one; as it returns, it writes the named tests and the time into `halt_named` and `halt_named_at`.
The row carries no `on_stuck`, the agent is no perch, and no review follows its fixes.

`DecisionHaltPrefix`'s doc comment, and its line in `shedadapters`' `doc.go`, are widened to name both producers that open a reason with it, the Bouncer's held-refusal halt and `Drift-Verify`'s decision halts, and that `arbiterstop` hands every such halt to the parent.
`pattern/PATTERN-gate-slot.md`'s verified-tree site list gains `Drift-Verify`'s gate, which runs its own `drift-verify` key's call through `Verify` and shares no pass with another site.
`doc.go` gains the `Drift-Verify` section.

`gateLoopSeats`, a test helper in the test file, wraps a `shedfake.SeatRunner` whose `RunFn` evaluates the table's gate entries in order as the run loop does at each turn end: it runs a scripted agent action (a commit, an `Accept`, dirt) before each arrival, re-arrives after a failure while the entry has attempts left, stops at a `Terminal` result or a spent entry, and returns a `seatengine.Result` whose chair carries a `GateOutcome` with each entry's state and attempts, as shuttle reports it.
`TestDriftVerify` (tier0) is a table over fake `Deps`, a fake `Compare` scripted per call, and `gateLoopSeats`:
no scope and scope `off` are Done with no compare; a waived scope is Done with no compare; no counted capture is a decision halt naming `lyx drift waive` under `partial` and `full`, in loom (passed with no `awaited_at`) and darn (no record); an undeclared compare is Stuck naming `lyx config drift`, with the prefix under `full`; a passing compare is Done with no agent; a failing compare spawns one agent and resets the run count; the agent fixing so the final compare passes is Done; a final compare still failing is Stuck naming `lyx drift accept`, without the prefix under `partial`, with it under `partial` with a withdrawn test and under `full`; a dirty tree re-prompts and, left dirty, is Stuck naming committing or cleaning, with the prefix under `full`; a verify entry failing its three attempts is Stuck naming its findings file, with the prefix under `full`; the verify entry does not run while HEAD equals the attempt head and runs once the agent commits, and a fix after its re-prompt is compared again before Done; an entry with a dirty tree is Stuck before any compare, and an entry with HEAD past the attempt head and a failing verify gate is Stuck before any compare; under `full` an agent acceptance with a passing compare is a decision halt naming the test and recording it, the parent's resume (a history Stuck whose `At` is at or after the recorded time) marks it `ruled` and a passing compare is Done with no agent, a folded Stuck whose `At` predates the recorded time and whose `LastAt` follows it is a resume too, a last history entry of another row or older than the recorded time rules nothing and halts again, a test withdrawn at the halt is not ruled and is compared again, and an agent that accepts and then errors leaves the resumed entry halting before any compare; `Commit` runs on every way out.

**ImpactSummary:** Three new `Deps` fields and one widened doc comment; the prefix's value and its readers are unchanged.
