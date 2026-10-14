# Card 4 — loop-status-unreadable

**Create:**
- `plan:internal/shedverbs#readLoopStatus` -> `func readLoopStatus(spec *Spec) (shedengine.Status, bool, error)`
- `plan:internal/shedverbs#statusStopError` -> `func statusStopError(spec *Spec, producer, cause string, decodeError bool) (string, string)`

**Edit:**
- `internal/shedverbs#runLoop`
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Uses:**
- `internal/shedverbs#interruptedError`
- `internal/loomcli#loomCLI.loomAfterInterrupt`

**Intent:** D5: the loop tells an absent status file from an unreadable one, and its stop points at its own log.

`LoopSpec` gains the field `ReadStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`, the loop's status reader; nil selects `state.ReadJSONNoMkdir[shedengine.Status]`.
It is a told test seam like `Runner` and `Now`, and lets a read error be injected that clears or persists.
`readLoopStatus` calls it on `spec.StatusPath` and `spec.StatusLockPath` and returns its found flag and error as they are.

At the top of each iteration `runLoop` reads through `readLoopStatus` instead of `readStatusFile`:
- Absent (no error, not found): the stop is interrupted with cause `status-missing`.
- A decode error (`errors.Is(err, state.ErrDecode)`): the stop is interrupted with cause `status-unreadable: <error>`, with no re-read, since the only writer, `internal/state`, writes by atomic rename under the status lock, so the file stays undecodable to this binary.
- Any other error, a read-lock or raw read failure: when the loop's single re-step budget (`restepped`) is unspent, it spends it, records `loop.restep` with the producer last read and the error, logs a `logger.Warn` and reads again; when the budget is spent, the stop is interrupted with cause `status-unreadable: <error>`.
  A re-read that succeeds steps on as before.
At each of these stops the loop writes a `logger.Warn` naming the status path and, when unreadable, the error; no child runs.
The cause keeps the `<class>: <detail>` form, as `start-failed: <error>` does, since `convergence-plan` classes halt reasons by their leading clause.

At these causes the trace file handed to `Hooks.AfterInterrupt` and written into the interrupted record is `spec.Loop.LogPath`, as `loop-exited` already does; the halt note's history stays empty, since loom's `loomAfterInterrupt` re-reads the file.
The `AfterInterrupt` field's doc on `Hooks`, which says it receives the child's first trace file, says it receives the loop's log instead at `loop-exited`, `status-missing` and `status-unreadable`.
Their message and way forward come from `statusStopError` instead of `interruptedError`, which keeps serving every other cause unchanged; the message keeps the `step <producer> interrupted (<cause>)` form.
Each way forward names the status path and the loop log, and is split by cause:
- `status-missing` names the recipe's own reseed: `spec.MissingStatusWayForward` with its leading `way forward: ` removed (`lyx loom start` for loom, `lyx batten run <slug>` for batten), and for an empty one `seed the run through its recipe's bootstrap verb, or "lyx shed seed" for a recipe without one`.
- `status-unreadable` from a read error names re-running `lyx shed step <run-id> --until-stop`.
- `status-unreadable` from a decode error names re-stepping with a binary that decodes the file by re-running `lyx shed step <run-id> --until-stop`, as after a version skew where an older loop binary cannot decode a file a current binary reads, and otherwise reporting the stop to the run's parent with the status path and the loop log.
At these causes `runLoop` skips `shedengine.WriteFailedStop`, so the status file is left as found, never written `failed`.
Bound: the status is then not made non-running, a stated departure from `PATTERN-failure-stop-visible` for a status that cannot be read, since overwriting it would destroy the evidence; a file that later reads `running` is resumed by the way forward's re-step, not by `lyx loom resume`, and `lyx loom arbiter` refuses either stop under its `status or envelope unreadable` row.
The loop's first read before the pid file and its final read after the stop stay on `readStatusFile`.

`TestLoop_InterruptedChildFillsTheErrorForm`'s subtest "status file vanishes between steps" becomes a table over the same one-continuing-child fixture, each row asserting exit 1, `kind: interrupted`, the cause in the error, exactly the expected number of children, the interrupted record's `trace_file` equal to `fx.spec.Loop.LogPath`, the trace file an `AfterInterrupt` hook received, the way-forward text, the status file's bytes after the stop, and, through `logger.SetOutput` capturing the process-global logger the file already runs unparallel for, a WARN line naming the status path:
- each row sets `fx.spec.Hooks.AfterInterrupt` to a recorder and, where it names one, `fx.spec.MissingStatusWayForward` on its own fixture, so `newLoopFixture` stays unchanged;
- each `ReadStatus` fake counts its calls and passes the first, the read before the first child, through to the real file, so the first child runs and the fault lands on the read after it;
- the child removes the status file: cause `status-missing`, way forward naming the told `MissingStatusWayForward` (the fixture sets one), the file still absent, one child;
- the same with no told clause: the way forward names `lyx shed seed`;
- the child overwrites the status file with `{not json`: cause `status-unreadable: `, way forward naming a binary that decodes the file and the run's parent, the bytes unchanged, one child and no `loop.restep`;
- `ReadStatus` set to a fake that fails twice with a raw read error after the first child: cause `status-unreadable: `, way forward naming `lyx shed step run-1 --until-stop`, `loop.restep` present, the status file still reading `running`, one child;
- `ReadStatus` set to a fake that fails once and then reads the real file, with a second child halting the run: the loop steps on, two children run and the stop is halted.

Docs in the same commit: `internal/shedverbs/doc.go`'s stop paragraph no longer says a vanished status file writes the run failed and states the two causes, their way forwards, the loop-log trace file and the departure; `contracts/specs/refusal-spec.md`'s shed `step interrupted` row drops "or the status file vanished", names `status-unreadable` from a read error that failed its re-read with the status file left as found and the re-step way forward, and a new row `loop status missing or undecodable`, class correctness halt like `status file missing`, names `status-missing` and a decode-error `status-unreadable`, the file left as found and each cause's way forward above; `pattern/PATTERN-failure-stop-visible.md` gains a bullet naming that departure, and its `## Enforcement` line names `TestLoop_InterruptedChildFillsTheErrorForm` for the status stops that leave the file as found.

**ImpactSummary:** The loop's per-iteration status read and its two status stops change inside `internal/shedverbs`; every other stop, the waiter's dead-loop path and loom's halt-note hook keep their behavior.

**Commit:** `4: loop-status-unreadable`
