```yaml
lens: way-forward
outcome: reported
```

I found six defects through the way-forward lens: one MEDIUM and five LOW. The refusal and stop paths I traced, and whether each one progresses:

- Card 14, refusal answer: `runRule` returns the snapshot and `*fabriccore.Refusal`, and the next step is the `Refusal.WayForward` text. It progresses only when that text is set; see finding 2.
- Card 14, failed action: the list ends and the rule is asked again with the outcome class. It progresses because the rule decides.
- Card 14, a `read` error: returned raw. It progresses for local reads; it does not for an origin transport failure (finding 1).
- Card 14, an action kind with no runner: a fail-loud wiring guard, which needs no next step. Partial effects are left behind (finding 4).
- Card 15, `ErrRecordsLayout`: passed up wrapped. Its message still names the refspecs to restore, so it progresses.
- Cards 2, 3 and 7: no new refusal. The existing refusals and way-forward text are kept word for word, including `branchStopWayForward` and the lock-dir messages.

Findings:

1. MEDIUM — Card 15 (`FieldOriginRefs`) with card 14 (`read` error path): an origin transport failure skips the core.
   - The reader reads origin through `lsRemoteHeads`, which just returns the error (`internal/fabricengine/originreconcile.go:542` and `:548`).
   - Card 15 wraps every read error, and card 14 returns a `read` error "as it is, a raw I/O failure", so the loop ends before the rule sees it.
   - Today `ReconcileOrigin` turns a transport failure into `OriginReport.Unreachable` with a nil error and nothing judged (`internal/fabricengine/doc.go` origin reconcile section). No core rule can express that over this reader and loop.
   - Part 2's first flow would either break D5 by erroring where today reports unreachable, or have to add an engine-side branch, which D3 forbids.
   - Fix: give a phase read an outcome the rule can read. For example, record an origin-unreachable fact (`OutcomeTransportFailure`, or `StoreRefs.OriginRead` false plus a class) in the snapshot and ask the rule again, instead of returning.

2. LOW — Card 12 (`Refusal`, `Answer`): a refusal can carry no way forward, and an empty refusal can be read as "done".
   - `Refusal.Error` adds `WayForward` only "when it is set", so the type the core uses for every rule admits a refusal with no next step.
   - `Answer` is "exactly one of" a refusal, a phase request or an action list, and an answer with none means done. The card does not say whether `Answer.Refusal` is a pointer.
   - If it is a value, a refusal whose `Reason` is empty reads as done, and `runRule` returns a nil error: the stop becomes a silent success.
   - Fix: make `Answer.Refusal` a `*Refusal`. Have the doc say every fabric refusal names its way forward, and add a `TestRunRule` row that a refusal with an empty reason still refuses.

3. LOW — Card 14 (re-read after a failed action): what a re-read failure keeps is not specified.
   - After a non-done outcome, the loop re-reads `Reread` before asking the rule again. A re-read failure then returns a raw error.
   - The card does not say that the returned snapshot still holds the failed action's outcome, so the caller may lose the class and its detail.
   - Fix: state that the returned snapshot keeps every appended outcome on a read error, and add a `TestRunRule` row for a failed action followed by a failed re-read.

4. LOW — Card 14 (wiring fault): a missing runner can stop a list after earlier actions already ran.
   - The no-runner error is raised when the list reaches that action.
   - Actions earlier in the same list have already run, so the verb stops halfway, and no outcome recording is specified for them.
   - Fix: check every action kind in a list against the runners before running any of them.

5. LOW — Card 15 (`TestSnapshotReader_RefStorage`): the files-store row does not name its fields.
   - The row reads "with `PATH` emptied, so the reader starts no git process".
   - `FieldOriginRefs` (`lsRemoteHeads`, which runs ls-remote) and possibly `FieldAnchors` (`committedMarker`) start git, so a row that names them fails with exec-not-found instead of passing.
   - Fix: name the fields the row reads (`FieldWorktrees` and `FieldLocalRefs`).

6. LOW — Card 15 (`FieldRecordsStore`): two error rules contradict each other.
   - One line says `ErrRecordsLayout` is returned "as it is"; another says every read error is "returned wrapped with the field and the store".
   - Fix: keep one rule. Wrapping with `%w` keeps `errors.Is` and the refspec way forward, so say that.

Tool calls:
- I grepped `lsRemoteHeads` in `internal/fabricengine/originreconcile.go` to see how it returns a transport failure, which decides whether finding 1 holds; my context lacked its body.
- I looked for `refusal-spec.md` under `_bolt/_lym/specs/*/` for the fail-loud guard rule. The grep found nothing there, so finding 4 rests on the card's own text.
