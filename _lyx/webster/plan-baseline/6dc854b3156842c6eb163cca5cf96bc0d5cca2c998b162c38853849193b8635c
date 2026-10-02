# Card 1 — GateSpec becomes an ordered list of gate entries

**Edit:**
- `internal/shuttleengine/gate.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/gate_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/burlerengine/engine.go`
- `internal/burlerengine/profile.go`
- `internal/burlerengine/engine_test.go`
- `internal/shedadapters/singlellm.go`
- `internal/shedadapters/singlellm_test.go`
- `internal/shedadapters/burler_test.go`
- `internal/shedrecipe/entries_gate.go`
- `internal/shedrecipe/fixture_test.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `internal/shedrecipe/entries_planwrite_test.go`
- `internal/shedrecipe/entries_prrework_test.go`
- `internal/shedrecipe/registry_test.go`
- `internal/shedbuild/fixture_test.go`
- `internal/loomrecipe/fixture_test.go`
- `internal/websterengine/runlevel.go`
- `internal/websterengine/runlevel_test.go`
- `internal/loomcli/smoke_gate_test.go`

**Create:**
- `internal/shuttleengine#GateEntry`
- `internal/shuttleengine/gate_list_test.go`

**Uses:**
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`
- `internal/shedadapters/gateattempts.go`

**Intent:**
Turn `GateSpec` into an ordered list of gate entries evaluated per arrival with per-entry consecutive-failure counts (decision record Decisions 1–7 and 9), and migrate every caller so the tree builds and every existing test keeps its assertions, bar the one `shedrecipe` default assertion named under Callers.

Types in `gate.go`:
`GateEntry` carries `Name string`, `Gate Gate`, `Attempts int` and `PassOnCap bool`;
`GateSpec` becomes `[]GateEntry`, and an empty or nil list means ungated.
Remove `defaultGateAttempts` and `GateSpec.attempts`: `Attempts` 0 means the entry is off, skipped at every arrival whatever its `PassOnCap`.
`GateOutcome` keeps `Passed`, `Attempts` and `FindingsPath` and gains a per-entry slice in list order, each element carrying the entry's name, its re-prompts sent, and its state at the final arrival.
The state is a small string-typed enum with five values: passed, failed, let through, off, not reached; the type and constant names are the implementer's call.
The `PassOnCap` doc comment states the limits from the decision record's Out list and Decision 4: counts live in memory for one shuttle run, so after an attach a capped entry starts from zero and can fire again;
a pass before the cap resets the count, so the entry can fire again later;
a request that must not repeat makes its own closure idempotent.

Run state in `run.go`: replace the single `gateSent` with per-entry counts, index-aligned with `run.gate`: consecutive failures and re-prompts sent.
The count slices are sized from `len(run.gate)` on first use, so neither `Run` literal (`start` in `run.go`, `reconstructAndWait` in `attach.go`) nor a test's bare `&Run{...}` needs a new field, and an attached gated run counts exactly as a started one.
`GateOutcome.Attempts` is the sum of re-prompts sent over all entries.
The memo stays per arrival and cleared before each re-prompt; it now also records which entry failed (none when every entry passed, was off, or was let through), so Wait can read it without re-running a closure.

Arrival evaluation (`Run.evaluateGate`): run the entries in list order and stop at the first failure.
An off entry is reported off; a `PassOnCap` entry whose failure count has reached `Attempts` is not run and is reported let through;
any other entry runs its closure and is reported passed or failed.
Every non-off entry after the stopping entry is reported not reached; an off entry is reported off wherever it sits.
A failing entry's findings are written to the one `gate-findings.md`, overwriting any earlier file, exactly as today.
`Passed` is true only when every entry that is neither off nor `PassOnCap` passed at this arrival;
`FindingsPath` is the failing entry's findings file when `Passed` is false, else empty.
A list of only off entries yields a non-nil outcome with `Passed` true.
A closure error is returned as today, stores no memo, and changes no count.
`evaluateGate` never changes a count — only Wait's re-prompt branch does — so `finalize` evaluating afresh after a lost session or a deadline cannot change one.

Wait's gated-Done branch in `wait.go` tests `len(run.gate) == 0` where it tested a nil `Gate`.
On an arrival with no failing entry, finalize.
On a failing entry whose failure count has reached its `Attempts` (only possible without `PassOnCap`), finalize; the outcome is the failed one the memo already holds.
Otherwise send `gateRepromptText` unchanged; a failed send finalizes as today.
After a successful send, increment the failing entry's failure count and sent count, reset to zero the failure count of every entry reported passed on this arrival, log the existing Warn line with the failing entry's name added and `budget` taken from that entry's `Attempts`, and clear the memo.
`finalize` keeps its single verdict site and its `fmt.Errorf("shuttle: gate: %w", ...)` wrap; the `completionsignal_enforcement_test.go` tripwire marker stays reserved for the two existing finalize call sites, so `evaluateGate` returns its errors unwrapped as today.
Update the package and method doc comments in `wait.go`, `run.go` and `gate.go` that describe the single gate, `gate.Gate`, or the default budget.

Callers:
`burlerengine.Engine.Run` builds a fresh `GateSpec` copy in which every entry's closure is wrapped by `repairReportBeforeGate`, never mutating the caller's slice;
off entries are wrapped too, since they never run.
`shedrecipe`'s `resolveGateSpec` keeps reading `gate`/`gate_attempts` in this card and returns a one-entry list named after the `gate` value, with `Attempts` 3 when `gate_attempts` is absent, zero or negative — today's `defaultGateAttempts` reading — so no interim value turns a gate off and the shipped recipe's behaviour is unchanged until Card 2 replaces the keys.
Because the default now resolves in `shedrecipe`, `entries_planwrite_test.go`'s `AbsentGateAttemptsYieldsPackageDefault` subtest changes its assertion from `Attempts` 0 to the single entry's `Attempts` 3, the one existing assertion this card changes rather than migrates;
the subtests asserting `Attempts` 0 for a row with no `gate` key (`entries_discussionwrite_test.go`, `entries_burler_test.go`) assert an empty list instead.
Every test fake's `RunGated`/`AttachGated` (in `burlerengine`, `shedadapters`, `shedrecipe`, `shedbuild`, `loomrecipe`) treats an empty list as ungated and otherwise runs the entries in list order, skipping off entries and stopping at the first failure, stamping the aggregate `GateOutcome` as it does today.
Test constructions of `GateSpec{Gate: g}` become one-entry lists with an explicit `Attempts` of 3 wherever the test relied on the removed default, and assertions reading `spec.Gate`/`spec.Attempts` read the single entry instead.
Doc comments that say "a nil Gate means ungated" in `websterengine/runlevel.go`, `burlerengine/profile.go` and `shedadapters/singlellm.go` say an empty list instead, and every other comment in this card's edited files that names `gate.Gate`, `opts.Gate.Gate` or a nil `Gate` (among them `burlerengine/engine.go`'s `RunGated` field and `Run` method comments, and `shedadapters/singlellm.go`'s field and `NewSingleLLMProducerGated` comments) describes the list.
`shedadapters/gateattempts.go` stays unchanged because the aggregate `Attempts` keeps its meaning.

Tests in the new `gate_list_test.go`, driven by the fake engine and scripted gate results the existing `gate_test.go` tests use:
- two entries, the first fails then passes: the second runs only on the arrival after the first passed;
- an entry that fails, passes, then fails again starts its budget afresh, both with and without `PassOnCap`;
- a final arrival (deadline, and failed re-prompt send) that stops at a failing `PassOnCap` entry with a non-`PassOnCap` entry after it finalizes `Passed` false with the later entry reported not reached;
- exhaustion without `PassOnCap` finalizes Done with `Passed` false and the failing entry's findings path;
- `PassOnCap` with `Attempts` 1: one re-prompt, then let through without its closure running again, `Passed` true, and still let through after a later entry's re-prompt;
- an `Attempts`-0 entry is never called, with and without `PassOnCap`, and is reported off;
- each of the five per-entry states appears where expected, and the aggregate `Attempts` is the sum of every entry's sent count;
- an empty list leaves `Result.Gate` nil, and a list of only off entries yields a non-nil `Result.Gate` with `Passed` true;
- a closure error on the second entry stays an infrastructure error, never a failed verdict.

`burlerengine/engine_test.go` gains a test that a failing second entry's findings carry the repair instruction naming the round's review and fixer-report paths, recovered from the wrapped closure the fake recorded, as the existing first-entry test does.

**ImpactSummary:** Changes the exported shape of `shuttleengine.GateSpec` and `GateOutcome`; eight packages read or build a `GateSpec` and are migrated here (`shuttleengine`, `burlerengine`, `shedadapters`, `shedrecipe`, `shedbuild`, `loomrecipe`, `websterengine`, `loomcli`), the pass-through callers (`webstercli`'s and `loomcli/cli.go`'s `StartMaster`, `attach.go`) compile unchanged, and single-entry behaviour is byte-identical.

**Commit:** 1: shuttle: carry an ordered list of gates, each with its own budget and pass_on_cap
