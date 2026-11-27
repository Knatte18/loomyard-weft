# Card 14 — executor-loop

**Create:**
- `internal/fabricengine#snapshotRead`
- `internal/fabricengine#actionRunner`
- `internal/fabricengine#fieldReadError`
- `internal/fabricengine#fieldReadError.Error`
- `internal/fabricengine#fieldReadError.Unwrap`
- `internal/fabricengine#runRule`
- `internal/fabricengine#outcomeClassOf`
- `internal/fabricengine#TestRunRule`
- `internal/fabricengine#TestOutcomeClassOf`

**Edit:**
- `cmd/lym#testOnlyCodeAllowed`
- `internal/fabricengine/livestate_doc_test.go`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/testkit/fabrickit#NewSnapshot`
- `internal/testkit/fabrickit#Builder.Build`
- `internal/fabriccore/doc.go`
- `internal/gitexec#Run`
- `internal/gitrepo#Repo.PushRebaseFree`

**Intent:** Add the engine's executor loop every later part drives (D3, D9 part 1), in `internal/fabricengine/executor.go`.

`fieldReadError` is the error a snapshot read returns for one field: it holds the `Field`, the `Store` and the inner `Err`.
`Error` names the field and the store and keeps `Err`'s message, and `Unwrap` returns `Err`, so `errors.Is` still finds a sentinel the inner error wraps.

`runRule` runs one verb's rule from an empty snapshot, treating `start` as the rule's first answer, and loops:
- An answer that sets more than one of a refusal, a phase request and an action list is a wiring fault: `runRule` returns an error naming the answer's parts, a fail-loud guard in `refusal-spec.md`'s sense, since only a rule this repository writes builds an answer.
- A refusal, a non-nil `Answer.Refusal`, ends the loop: `runRule` returns the snapshot and the `*fabriccore.Refusal` as its error, whatever its reason holds.
- A phase request calls `lock` with the phase at that phase's first request only, then calls `read` with the phase and its `Read` fields, appends the phase to `Snapshot.Phases` once that read succeeds, and asks the rule again;
  a phase whose read failed enters `Phases` only when a later request of it reads successfully.
  A later request of a phase already locked reads under the lock still held, so a rule that asks for a phase again after a read failure never takes the verb's lock twice.
  So a verb's `lock` takes the lock it takes today once the under-lock phase is asked for (D2), and answers a phase that needs no lock with a release that does nothing.
- An action list first checks that every action's kind but `ActionWait` has a runner; a kind with none is a wiring fault, returned as an error naming the kind before any action of the list runs, since the engine registers each runner beside the rule that emits its kind.
  It then runs its actions in order through the runner its kind names.
  `ActionWait` needs no runner: the loop waits out `Action.Delay` through `sleep`, and a nil `sleep` sleeps for real.
  Each action's outcome, its class from `outcomeClassOf` and its error text, is appended to `Snapshot.Outcomes`.
  The first outcome that is not `OutcomeDone` ends the list, so no later action of it runs (D3).
  Once the list ends, `read` re-reads the answer's `Reread` fields under the latest phase, the phase requested last whether or not its read succeeded, or the zero `Phase` when none was requested yet, and under that phase's lock, still held (D3), and the loop asks the rule again.
- `runRule` holds every release `lock` returned until it returns, on every return path, a refusal, an error or the end, and calls them in reverse order.
  A nil `lock` takes none, and a `lock` error is returned with the snapshot as it stands, as a raw read error is, with no read of that phase.
- A `read` error, at a phase request or at a re-read, is taken apart into the `*fieldReadError`s it holds: through `errors.As` on it, or, for an error joining several, on each error its `Unwrap() []error` returns.
  When `outcomeClassOf` of every one's `Err` is a transport failure, one `fabriccore.ReadFailure` per `*fieldReadError` is appended to `Snapshot.ReadFailures`, in the joined order, its `Phase` the requested phase, or for a re-read the phase the re-read ran under, its `Field` and `Store` from the error, its class from `outcomeClassOf` and its text from `Err`, and the loop asks the rule again, so the rule decides what an unreachable origin means (D3).
  Any other read error, a joined one holding an error that is no transport failure, and a read error that is no `*fieldReadError`, is returned as it stands with the snapshot, every outcome and read failure appended so far kept, a raw I/O failure.
- An answer with nothing in it ends the loop with the snapshot and a nil error: the rule alone decides when the loop stops (D3).

`outcomeClassOf` maps an action's or a read's error to its class, the one rule the executor keeps (D3):
nil is done; an `*AnchorStopError` an anchor stop; `gitrepo.ErrPushAuthRefused` auth refused; `gitrepo.ErrPushRejected` rejected; a `destructiveRefusal` from `destroy.go` a destructive refusal; `gitexec.IsTransportFailure` a transport failure; anything else failed.
Today's `rejectionRetryKind` stays as it is until part 2 moves the push concern onto the loop.

The runner of every other kind lands with the first part whose rule emits it, as D6's one tier1 test per action kind does; this card registers none.

`runRule`, `outcomeClassOf`, `fieldReadError.Error` and `fieldReadError.Unwrap` have no production caller until part 2, `fabric-core-push-origin`.
So `cmd/lym#testOnlyCodeAllowed` admits `internal/fabricengine.runRule`, `internal/fabricengine.outcomeClassOf`, `internal/fabricengine.fieldReadError.Error` and `internal/fabricengine.fieldReadError.Unwrap`, as the added entry "A chain part may leave production code that only a named later part first calls" allows.
Each entry's reason names `TestRunRule` or `TestOutcomeClassOf` and `fabric-core-push-origin` as the part that adds its first production caller and removes the entry in that change.
`testOnlyCodeAllowed`'s doc comment names this second, temporary kind of entry beside the shared reference or helper:
a function of a `fabric-core` chain part that only tests reach until a named later part adds its first production caller, whose reason names that part's slug, and which that part removes in the change that adds the caller.

Tests, tier1, in the internal package, with snapshots from `fabrickit`.
Every `gitexec` or `gitrepo` error they use is provoked in the real module against local git (`PATTERN-test-stand-ins`):
- transport: the error of `gitexec.Run` running `ls-remote` against an origin URL that refuses the connection;
- rejected: the error of `gitrepo`'s `PushRebaseFree` pushing a branch whose origin tip diverged;
- auth refused: the error of `PushRebaseFree` pushing to a bare repository whose pre-receive hook prints `Permission to hub denied to tester.`.

Errors of the engine's own types, `*AnchorStopError`, `destructiveRefusal` and a `*fieldReadError` wrapping a provoked error, and plain `errors.New` values are built in process.
- `TestRunRule` (table) drives `runRule` with a rule each row writes over the snapshot, a `read` that fills the named fields from a `fabrickit` snapshot or returns the row's error, a `lock` that records each phase it is called with and each release, and runners that return each row's error.
  Rows:
  a start refusal whose reason is empty is returned as the error, with no read and no lock;
  a phase request takes its phase's lock, reads its fields and records the phase, an action list whose actions all succeed then runs them in order and re-reads the named fields under that phase, and the empty answer that follows ends the loop with a nil error and the lock released;
  an action list as the start answer re-reads under the zero phase;
  a failed action ends its list, skips the rest and hands its outcome class to the rule;
  a wait action sleeps the delay through the injected sleep;
  a phase read failing with two joined `*fieldReadError`s, one per store, each wrapping a provoked transport error, is recorded as two read failures in that order, each with its phase, field, store, class and text, and the rule is asked again, and its second request of that phase reads again under the one lock the first request took, `Phases` holding that phase once, from the second read;
  a failed action followed by a re-read failing with another error returns that error with the failed action's outcome kept in the snapshot;
  an under-lock phase request takes the lock before its read, and a refusal that follows returns with the lock released;
  a lock error is returned with no read.
  Each row asserts the returned snapshot's phases, outcomes and read failures, the runner, lock and release calls in order and the error, as literals,
  but for a provoked error's text, which a row wants as that error's own message, since git's wording and the temporary paths it names are not the engine's to pin.
- `TestOutcomeClassOf` (table): one row per class: nil is done, the provoked transport, rejected and auth-refused errors take their classes, an `*AnchorStopError` and a `destructiveRefusal` theirs, and an `errors.New` value is failed.

The two wiring faults take no row: each guards the engine's own registration, not an input.

`doc.go` gains a section on the executor loop: the snapshot and action list vocabulary (D11), the loop's steps above, the lock seam, the read failures handed to the rule, the error-to-class mapping as the engine's one rule, and the runners landing with each part.
The section says its "snapshot" is the core's input, distinct from the snapshot tags the `Snapshot:` trailer records and from the recorder's `Snapshot`.
`livestate_doc_test.go`'s naming section names both meanings: the `Snapshot: <tag>` trailer, and the core's input snapshot, which the harness does not drive.

**ImpactSummary:** New unexported code in one engine file that no verb calls yet, a doc section, one harness doc sentence and the dead-code allowlist entries for the loop; no existing behavior changes.

**Commit:** `14: executor-loop`
