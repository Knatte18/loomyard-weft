# Card 4 — adapter-routes-conflict

**Create:**
- `plan:internal/websterengine#ConflictRoute` -> `type ConflictRoute struct`
- `plan:internal/websterengine#ClaimConflictRoute` -> `func ClaimConflictRoute(scratchDir string) (ConflictRoute, error)`
- `plan:internal/shedadapters#planConflictWayForward` -> `const planConflictWayForward = "edit the named pending cards by hand so they match the tree, then run \"lyx loom resume\" in the task worktree"`
- `plan:internal/shedadapters#TestWebsterProducer_ConflictRerouteCapThenRepeatHalts` -> `func TestWebsterProducer_ConflictRerouteCapThenRepeatHalts(t *testing.T)`

**Edit:**
- `internal/shedadapters#WebsterProducer.Call`
- `internal/shedadapters#TestWebsterProducer_OutcomeStuck`
- `internal/shedadapters#TestWebsterProducer_OtherEngineErrors`
- `internal/shedadapters#TestWebsterProducer_CancelledDuringRun_StuckPausedAndErrorBecomeContextError`
- `internal/shedadapters#TestWebsterProducer_NoBridgeInstalled`
- `internal/loomshed#TestWebsterProducer_Call`
- `internal/loomshed#TestWebsterProducer_PassesStuckReasonThrough`
- `internal/shedadapters/doc.go`
- `contracts/specs/refusal-spec.md`
- `internal/websterengine/doc.go`

**Intent:**
Decision D3's producer half and D5's loom rows: a Webster stuck outcome that follows a new plan conflict reroutes to plan review; a repeated one halts.

In `internal/websterengine/conflictrecord.go`, `ConflictRoute` carries `Found bool`, `New bool` and `Findings []string`.
`ClaimConflictRoute` reads the conflict record at `ConflictRecordPath(scratchDir)`;
absent, or present with no `- ` finding line (a torn write), it returns the zero `ConflictRoute` and nil, so a record naming no finding never claims a recurrence.
Otherwise it takes the record's `- ` finding lines and compares them, as a set (order ignored, batch heading and cause left out), with the finding lines of the routed file at `RoutedConflictPath(scratchDir)`, an absent routed file counting as the empty set.
When they differ it writes the record's findings to the routed file in the same `- <finding>` line form and returns `Found` and `New` true;
when they are equal it returns `Found` true and `New` false and writes nothing.
A read or write error is returned.

`WebsterProducer.Call`'s `websterOutcomeStuck` arm, after its existing `cancelErr` check, calls `websterengine.ClaimConflictRoute(p.deps.Geom.ScratchDir)`:
- `New`: log Warn and return `Stuck` with `OutputPointer{Reroute: true, Reason: ...}`, the reason `webster plan conflict, asking a reroute to plan review: <stuck reason>; if plan review does not clear it: ` + `planConflictWayForward`, worded so a capped reroute's halt, which quotes it, claims no review took place;
- `Found` and not `New`: return plain `Stuck` with the reason `the same webster plan conflict recurred after a reroute of it was asked for: <stuck reason>; way forward: ` + `planConflictWayForward`, which claims no plan review took place, and halts, the row having no `on_stuck`;
- not `Found`: return `Stuck` with the stuck reason as today;
- an error: log Warn and return `Stuck` with the stuck reason as today, so a run whose record or routed file cannot be read or written halts rather than reroutes.
The routed file is written at verdict time, before shed decides whether the reroute is taken, so a capped reroute counts as routed and resuming its halt without the hand edit halts as a repeat.
The error arms, `ErrAutoRebaseline` and `ErrPendingAuditFindings` included, never consult the record and never reroute.
The bound on what the new routing lets through: a reroute needs a stuck outcome, a record from this Webster call (card 2 removes it at Run entry and every begin-batch), and findings unlike the last routed ones, so one conflict earns at most one plan review while its card ids are unchanged, and shed's cap of three bounds distinct conflicts in a row.
`internal/shedadapters/doc.go`'s WebsterProducer bullet and `internal/websterengine/doc.go`'s conflict-record section describe the claim and the three stuck outcomes.

`contracts/specs/refusal-spec.md`'s loom section gains three rows:
"Webster: plan conflict rerouted" (a Webster stuck outcome with a conflict record whose findings differ from the last routed ones; not a halt, the run moves to Plan-Burler-Review; way forward: none needed, plan review edits the pending cards and the main line returns to Webster);
"Webster: plan conflict repeated" (the same findings recur after a reroute of them was asked for; correctness halt; edit the named pending cards by hand so they match the tree, then `lyx loom resume`);
"Webster: plan conflict reroute capped" (shed's reroute cap halts a Webster reroute; correctness halt; the same hand edit, then `lyx loom resume`, as the told clause `run "lyx loom resume" in the task worktree once the cause the halt's reason quotes is cleared` card 3 gives `loomRerouteCapWayForward` says).

Tests:
every adapter test whose run reaches the stuck arm builds its `RunDeps` with `Geom.ScratchDir` set to a temp dir, so no claim resolves against the test's working directory: `TestWebsterProducer_OutcomeStuck`, `TestWebsterProducer_NoBridgeInstalled`, `TestWebsterProducer_CancelledDuringRun_StuckPausedAndErrorBecomeContextError`, `TestWebsterProducer_OtherEngineErrors`, and `internal/loomshed`'s `TestWebsterProducer_Call` and `TestWebsterProducer_PassesStuckReasonThrough`, whose stuck rows drive the same arm through `loomshed`'s wrapper.
`TestWebsterProducer_OutcomeStuck` becomes a table over `NewWebsterProducer` and `fakeWebsterRunner` returning a stuck outcome, with a scratch dir per row seeded through `WriteConflictRecord` and an optional pre-written routed file; its existing case is the no-record row, plain `Stuck` with the stuck reason.
New rows:
- a new record with no routed file returns `Reroute` with a reason naming `lyx loom resume` and the pending cards and not claiming a review took place, and leaves the routed file holding the findings;
- a record whose findings equal the routed ones, and one whose findings are the routed ones reordered, each return plain `Stuck` with the repeated reason, no "review" claim, and the routed file unchanged;
- a record whose findings differ from the routed ones reroutes and rewrites the routed file;
- a record with a heading and no finding line returns plain `Stuck` with the stuck reason;
- a record path that is a directory, and a new record whose routed path is a non-empty directory, each return plain `Stuck` with no `Reroute`, the directory left in place.
`TestWebsterProducer_OtherEngineErrors` seeds a conflict record in every row's scratch dir and its `AutoRebaselineWrap` row asserts `Reroute` is false and the routed file absent.
`TestWebsterProducer_CancelledDuringRun_StuckPausedAndErrorBecomeContextError`'s `Stuck` row seeds a new record and asserts the routed file stays absent, so a claim made before the cancellation check fails it.
The new `TestWebsterProducer_ConflictRerouteCapThenRepeatHalts` builds a `shedengine.Shed` (as `TestWebsterProducer_PendingAuditFindingsBlocksRun` does) with a `Webster` row whose `OnReroute` names a fake Done row, history seeded with three Webster `OutcomeReroute` entries, a told `RerouteCapWayForward`, a seeded record and a stuck runner:
the first `Run` halts blocked with a reason holding `reroute cap reached` and `lyx loom resume` with the pending-card hand edit, and the routed file holds the findings;
a second `Run` with the record unchanged halts blocked with the repeated reason and adds no `OutcomeReroute` entry.

**ImpactSummary:** Changes only the stuck arm of the one Webster adapter, which only the loom recipe's `Webster` row reaches, through `loomshed`'s wrapper; a stuck outcome with no conflict record maps exactly as before, six existing adapter tests gain a told scratch dir, and the Webster row's new `on_reroute` from card 3 is what the reroute takes.
