# Card 14 — executor-loop

**Create:**
- `plan:internal/fabricengine#snapshotRead` -> `type snapshotRead func(phase fabriccore.Phase, fields []fabriccore.Field, snapshot *fabriccore.Snapshot) error`
- `plan:internal/fabricengine#actionRunner` -> `type actionRunner func(action fabriccore.Action) error`
- `plan:internal/fabricengine#runRule` -> `func runRule(rule fabriccore.Rule, start fabriccore.Answer, read snapshotRead, runners map[fabriccore.ActionKind]actionRunner, sleep func(time.Duration)) (fabriccore.Snapshot, error)`
- `plan:internal/fabricengine#outcomeClassOf` -> `func outcomeClassOf(err error) fabriccore.OutcomeClass`
- `plan:internal/fabricengine#TestRunRule` -> `func TestRunRule(t *testing.T)`
- `plan:internal/fabricengine#TestOutcomeClassOf` -> `func TestOutcomeClassOf(t *testing.T)`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/testkit/fabrickit#NewSnapshot`
- `plan:internal/testkit/fabrickit#Builder.Build`
- `internal/fabriccore/doc.go`

**Intent:** Add the engine's executor loop every later part drives (D3, D9 part 1), in `internal/fabricengine/executor.go`.

`runRule` runs one verb's rule from an empty snapshot, treating `start` as the rule's first answer, and loops:
- An answer that sets more than one of a refusal, a phase request and an action list is a wiring fault: `runRule` returns an error naming the answer's parts, a fail-loud guard in `refusal-spec.md`'s sense, since only a rule this repository writes builds an answer.
- A refusal, a non-nil `Answer.Refusal`, ends the loop: `runRule` returns the snapshot and the `*fabriccore.Refusal` as its error, whatever its reason holds.
- A phase request calls `read` with the phase and its `Read` fields, appends the phase to `Snapshot.Phases`, and asks the rule again.
- An action list first checks that every action's kind but `ActionWait` has a runner; a kind with none is a wiring fault, returned as an error naming the kind before any action of the list runs, since the engine registers each runner beside the rule that emits its kind.
  It then runs its actions in order through the runner its kind names.
  `ActionWait` needs no runner: the loop waits out `Action.Delay` through `sleep`, and a nil `sleep` sleeps for real.
  Each action's outcome, its class from `outcomeClassOf` and its error text, is appended to `Snapshot.Outcomes`.
  The first outcome that is not `OutcomeDone` ends the list, so no later action of it runs (D3).
  Once the list ends, `read` re-reads the answer's `Reread` fields under the latest phase, the zero `Phase` when none was read yet, and the loop asks the rule again.
- A `read` error, at a phase request or at a re-read, goes through `outcomeClassOf`:
  a transport failure is appended to `Snapshot.ReadFailures` with its phase, its field and its error text, and the loop asks the rule again, so the rule decides what an unreachable origin means (D3);
  any other read error is returned with the snapshot as it stands, every outcome and read failure appended so far kept, a raw I/O failure.
  The reader wraps its errors with the field and the store, so the read failure's field is the first of the request's fields the error names, or the request's first field.
- An answer with nothing in it ends the loop with the snapshot and a nil error: the rule alone decides when the loop stops (D3).

`outcomeClassOf` maps an action's or a read's error to its class, the one rule the executor keeps (D3):
nil is done; an `*AnchorStopError` an anchor stop; `gitrepo.ErrPushAuthRefused` auth refused; `gitrepo.ErrPushRejected` rejected; a `destructiveRefusal` from `destroy.go` a destructive refusal; `gitexec.IsTransportFailure` a transport failure; anything else failed.
Today's `rejectionRetryKind` stays as it is until part 2 moves the push concern onto the loop.

The runner of every other kind lands with the first part whose rule emits it, as D6's one tier1 test per action kind does; this card registers none.
`runRule`, `outcomeClassOf` and the engine's call of `fabriccore.Refusal.Error` have no production caller until part 2, so `cmd/lym#testOnlyCodeAllowed` admits `internal/fabricengine.runRule`, `internal/fabricengine.outcomeClassOf` and `internal/fabriccore.Refusal.Error`.
Each entry's reason names `TestRunRule`, `TestOutcomeClassOf` or `TestRefusal_Error` and part 2 as the first caller; the overview's Shared Decisions record the departure.

Tests, tier0, in the internal package, with snapshots from `fabrickit`:
- `TestRunRule` (table) drives `runRule` with a rule each row writes over the snapshot, a `read` that fills the named fields from a `fabrickit` snapshot or returns the row's error, and runners that return each row's error, standing in for git.
  Rows:
  a start refusal returns it with no read;
  a refusal with an empty reason still returns it as the error;
  a phase request reads its fields and records the phase;
  an action list whose actions all succeed runs them in order, re-reads the named fields under the latest phase and asks again;
  an action list as the start answer re-reads under the zero phase;
  a failed action ends its list, skips the rest and hands its outcome class to the rule;
  a wait action sleeps the delay through the injected sleep;
  a phase read failing with a transport error is recorded as a read failure and the rule is asked again;
  a phase read failing with another error is returned;
  a failed action followed by a failing re-read returns the error with the failed action's outcome kept in the snapshot;
  an empty answer ends the loop.
  Each row asserts the returned snapshot's phases, outcomes and read failures, the runner calls in order and the error, as literals.
- `TestOutcomeClassOf` (table): one row per class, each error built in process, such as a `*gitexec.GitError` whose `Args` begin with `push` and whose stderr `IsTransportFailure` reads as a transport failure, and a wrapped `gitrepo.ErrPushRejected`.

The two wiring faults take no row: each guards the engine's own registration, not an input.

`doc.go` gains a section on the executor loop: the snapshot and action list vocabulary (D11), the loop's steps above, the read failures handed to the rule, the error-to-class mapping as the engine's one rule, and the runners landing with each part.

**ImpactSummary:** New unexported code in one engine file that no verb calls yet, a doc section and three dead-code allowlist entries; no existing behavior changes.

**Commit:** `14: executor-loop`
