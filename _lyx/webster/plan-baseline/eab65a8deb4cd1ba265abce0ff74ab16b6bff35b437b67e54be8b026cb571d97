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
- `pattern/PATTERN-gate-slot.md`
- `internal/shedadapters/doc.go`
- `internal/loomshed/doc.go`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `plan:internal/driftengine#VerifyTable`
- `internal/driftengine#RunCompare`
- `internal/driftengine#CountedCapture`
- `internal/driftengine#ResetRuns`
- `internal/driftengine#AttemptHead`
- `internal/driftengine#AgentSessionOpen`
- `internal/driftengine#CloseAgentSession`
- `internal/driftengine#DirRel`
- `internal/driftengine#ScopeRecord.Unruled`
- `internal/driftengine#UpdateScope`
- `internal/shedadapters#NewMultiLLMProducerSourced`
- `internal/loomshed#NewVerifyGate`
- `internal/testkit/shedfake#SeatRunner`

**Intent:**
Add `Drift-Verify`: the mechanical compare, then one strong agent on drift, and the decision halts that put drift in a task meant to change nothing to the parent (D6, D7).

`Deps` gains `VerifyTable func(told VerifyTold) (seatengine.Table, error)`, `Compare func(ctx context.Context) (CompareResult, error)` (built by the wiring over `RunCompare` with the scope's `SkipList`) and `GateFindingsFile string`.
`VerifyTold` carries the failing compare's log and the baseline's commit (the counted capture's `head`), which the table renders.
`verify` is the row's resolved `verify` gate entry, the `drift-verify` key's call through `loomshed.NewVerifyGate` and `verifytree.Verify`, built by the registry entry (card 14-drift-recipe-rows).

`DriftVerify.Call` commits all of `DirRel()` (report, scope record, logs, any capture log still uncommitted) through `Commit` on every way out, and in order:
1. no scope record, or scope `off`, is Done naming that reason; a waived scope is Done without a compare.
2. While `AgentSessionOpen` reads open, this entry may follow an interrupted agent: `Call` builds the verify table, told the newest compare log under `LogsDir` and the counted capture's `head`, and calls `Deps.Seats.Probe` on it.
   A live chair skips steps 3 to 8 and goes straight to step 9, whose `NewMultiLLMProducerSourced` resumes the live chair, so its fresh-spawn preparation and `ResetRuns` do not run; that keeps `InterruptPolicyReinvoke`'s premise, a re-step reattaching to a live agent before anything else (card 14-drift-recipe-rows).
   A closed mark, or a probe finding no live chair, closes the mark through `CloseAgentSession` and goes on.
3. Under `full`, when the scope's `halt_named_at` is set and `Status`'s last history entry is a Stuck of this row whose latest time (`LastAt` when set, `At` otherwise) is at or after it, this entry is the resume from that halt: every named test still on `accepted` is marked `ruled`, and `halt_named` and `halt_named_at` are cleared in the same `UpdateScope` call, before any compare.
   "At or after" parses both RFC 3339 strings, the record's written from `Deps.Now().UTC().Format(time.RFC3339)` (card 2-drift-records), and compares them as instants at second precision, since the shed stamps its history in that format after `Call` returns.
4. Under `full`, any `unruled` acceptance ends the row in a decision halt before any compare (below).
   So an agent session that accepts and ends in an error, transient or interrupted stop leaves an acceptance no halt recorded, and the resumed entry, a tackler's included, halts for the parent instead of comparing.
5. No counted capture (`CountedCapture` with `AwaitRow`) is Stuck with a reason opening with `shedadapters.DecisionHaltPrefix` under every scope, naming `lyx drift waive --reason <text>` followed by `lyx loom resume`.
6. `Config().Step("compare")` refusing is Stuck naming `lyx config drift` then `lyx loom resume`.
7. The handoff's first two checks again, so a resume from a dirty-tree or test-gate halt never compares past what it guarded (D6's order on every way out).
   A dirty tracked tree is Stuck naming the paths and committing or cleaning them, then `lyx loom resume`, on every entry that reaches this step.
   The dirty tree is the code checkout's tracked paths; records-side paths such as `_lyx/drift/scope.json` never count, since an agent's `lyx drift accept` and steps 2 and 3 write them before the check.
   Each of this step's halts, and step 10's exhausted-entry Stucks, writes `gate_halt_at` (the time, formatted as step 3's `halt_named_at` is) through `UpdateScope` before it returns, only when it is unset: a resume's halt keeps the time it read, so an interrupt between that halt's return and the shed persisting its Stuck leaves the next entry still the resume, and the new Stuck, stamped later, keeps the following resume true.
   The halts' opening texts are package constants that open those Stuck reasons, after `DecisionHaltPrefix` where it applies, for the operator's reading; no code reads them back.
   An entry is the resume from such a halt when `gate_halt_at` is set and `Status`'s last history entry is a Stuck of this row whose latest time (`LastAt` when set, `At` otherwise) is at or after it, the history test step 3 uses, at second precision.
   Every entry decides this once, right after step 2, and an entry that is not such a resume clears `gate_halt_at` there, before any halt, so a mark left by a gate halt the run moved past without a resume (a goto, or an interrupt before the shed persisted the Stuck) makes no later entry a resume.
   Such a resume keeps `gate_halt_at` set while the dirty check and the told `verify` entry's gate run, and runs that gate once, only when `AttemptHead` is found and differs from HEAD; no entry that is not such a resume runs it, so a first entry (`AttemptHead` absent) and a reroute entry (last history entry another row's) run no verify gate, since their commits are no `Drift-Verify` session's (D6).
   It clears `gate_halt_at` in one `UpdateScope` call only once the dirty check passed and the gate either passed or was not due (`AttemptHead` absent or equal to HEAD), so a gate ended by a context error, transient stop or interrupt, which records no history entry, leaves the next entry still the resume, which runs the gate again before any compare.
   A resume that halts at step 4, 5 or 6 leaves `gate_halt_at` set too, so the following entry is still the resume from the gate halt and still runs the gate once.
   A failure is Stuck naming its findings file and fixing the tests, committing, then `lyx loom resume`, and keeps `gate_halt_at` set, as above (a tree its record already passed skips without running).
8. The mechanical compare, which counts no run: a pass is Done and spawns nothing.
9. A failing compare spawns one agent through `shedadapters.NewMultiLLMProducerSourced` over `Seats` and `VerifyTable`, whose fresh-spawn hook calls `ResetRuns` with HEAD, so each fresh attempt has `verify_max_runs` runs; each resume reaching a failing compare spawns a fresh agent, bounded by the run's tackler and arbiter spawn budgets (D6).
   The table's gate is, in order: `dirty` (3 attempts, failing with the code checkout's tracked dirty paths, never a records-side one, and asking the session to commit them); the told `verify` entry, wrapped to pass without running while HEAD equals `AttemptHead`, so it runs only once the session committed code; and `compare` (one attempt, must-pass), which runs `Compare` and on a failure returns a `Terminal` result naming the log, so no fix the agent or a gate re-prompt makes goes uncompared.
10. The agent's end closes the mark through `CloseAgentSession`; an error is returned as the seat producer maps it (an error or transient stop, never settled with an acceptance); a Done is re-checked as in step 4 under `full`, else Done; a Stuck after the compare's terminal failure names the report and the ways forward, `lyx drift accept <test> --reason <text>` followed by `lyx loom resume`, or fix and `lyx loom resume`; a Stuck after the dirty or verify entry spent its attempts keeps the gate's findings file, has its reason opened with that entry's halt opening text from step 7 ahead of the seat producer's gate reason, writes `gate_halt_at` as step 7 says, and names its way forward, committing or cleaning the tree, or fixing the tests and committing, then `lyx loom resume`, whose entry re-runs step 7.
The producer tells the compare's terminal failure from an exhausted entry by the chair's `GateOutcome.Entries`, the entry in state `failed`.

Every `Drift-Verify` Stuck under `full`, and under `partial` the no-counted-capture halt and a final-compare failure while `withdrawn` is non-empty, has its reason opened with `DecisionHaltPrefix`, so `arbiterstop.HandsBack` hands it to the parent, ahead of the tackler and the arbiter (D6).
Every decision halt under `full` reached while an `unruled` acceptance stands names those tests and the parent's ways forward for them beside its own: `lyx loom resume` to stand by them, after any `lyx drift accept` of the parent's own, or `lyx drift withdraw <test>` and `lyx loom resume` to reject one; as it returns, it writes the named tests and the time into `halt_named` and `halt_named_at`.
The row carries no `on_stuck`, the agent is no perch, and no review follows its fixes.

`DecisionHaltPrefix`'s doc comment, and its line in `shedadapters`' `doc.go`, are widened to name both producers that open a reason with it, the Bouncer's held-refusal halt and `Drift-Verify`'s decision halts, and that `arbiterstop` hands every such halt to the parent.
`pattern/PATTERN-gate-slot.md`'s verified-tree site list gains `Drift-Verify`'s gate, which runs its own `drift-verify` key's call through `Verify` and shares no pass with another site.
`internal/loomshed/doc.go`'s verify-gate sentence, which names each row's own key, gains the `drift-verify` key at `Drift-Verify`.
`doc.go` gains the `Drift-Verify` section.

`gateLoopSeats`, a test helper in the test file, wraps a `shedfake.SeatRunner` whose `RunFn` evaluates the table's gate entries in order as the run loop does at each turn end: it runs a scripted agent action (a commit, an `Accept`, dirt) before each arrival, re-arrives after a failure while the entry has attempts left, stops at a `Terminal` result or a spent entry, and returns a `seatengine.Result` whose chair carries a `GateOutcome` with each entry's state and attempts, as shuttle reports it.
`TestDriftVerify` (tier0) is a table over fake `Deps`, a fake `Compare` scripted per call, and `gateLoopSeats`:
no scope and scope `off` are Done with no compare; a waived scope is Done with no compare; no counted capture is a decision halt naming `lyx drift waive` under `partial` and `full`, in loom (passed with no `awaited_at`) and darn (no record); an undeclared compare is Stuck naming `lyx config drift`, with the prefix under `full`; a passing compare is Done with no agent; a failing compare spawns one agent and resets the run count; the agent fixing so the final compare passes is Done; a final compare still failing is Stuck naming `lyx drift accept`, without the prefix under `partial`, with it under `partial` with a withdrawn test and under `full`; a dirty tree re-prompts and, left dirty, is Stuck naming committing or cleaning, with the prefix under `full`; a verify entry failing its three attempts is Stuck naming its findings file, with the prefix under `full`; the verify entry does not run while HEAD equals the attempt head and runs once the agent commits, and a fix after its re-prompt is compared again before Done; an entry with a dirty tree is Stuck before any compare; an entry with only a records-side path dirty is no dirty Stuck; a dirty-tree halt, a test-gate halt and an exhausted dirty or verify entry each record `gate_halt_at`; a resume from the row's test-gate halt (a seeded `gate_halt_at` and a last history entry that is a Stuck of this row at or after it) with HEAD past the attempt head and a failing gate is Stuck before any compare and keeps the seeded `gate_halt_at`, and so is a resume after the agent's verify entry spent its attempts, seeded the same way; a resume whose gate passes clears `gate_halt_at` and goes on to the compare; a resume whose verify gate returns a context error with HEAD past the attempt head leaves `gate_halt_at` set, and the next entry runs the gate again before any compare; a resume from the row's own dirty-tree halt with `AttemptHead` equal to HEAD does not call the told verify entry, clears `gate_halt_at` and goes on to the compare; with a fake `Now` returning a sub-second time and a seeded history `At` that is the same instant truncated to the second, the entry is still read as the resume, for `gate_halt_at` and for `halt_named_at`; with `gate_halt_at` set, a last history entry of another row (a reroute) or one older than `gate_halt_at`, with HEAD past the attempt head, runs no verify gate and goes on to the compare, and so does a first entry with no attempt head; a mark left by a gate halt the run left by a goto is cleared by the next entry, which is no resume, so that entry's no-counted-capture halt resumed with HEAD past the attempt head runs no verify gate; with the agent-session mark open and `gateLoopSeats` scripting a live chair for both the step-2 probe and the seat producer's own probe, the producer resumes it, and no dirty check, verify gate or mechanical compare runs before it; with the mark open and no live chair the mark is closed and the entry goes on to the compare; the mark reads closed after the agent's end, Done, Stuck or error alike; under `full` an agent acceptance with a passing compare is a decision halt naming the test and recording it, the parent's resume (a history Stuck whose `At` is at or after the recorded time) marks it `ruled` and a passing compare is Done with no agent, a folded Stuck whose `At` predates the recorded time and whose `LastAt` follows it is a resume too, a last history entry of another row or older than the recorded time rules nothing and halts again, a test withdrawn at the halt is not ruled and is compared again, and an agent that accepts and then errors leaves the resumed entry halting before any compare; `Commit` runs on every way out.

**ImpactSummary:** Three new `Deps` fields and one widened doc comment; the prefix's value and its readers are unchanged.
