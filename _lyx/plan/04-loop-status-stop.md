# Card 4 — loop-status-stop

**Create:**
- `plan:internal/shedverbs#readLoopStatus` -> `func readLoopStatus(spec *Spec) (st shedengine.Status, cause string, readFailed bool)`
- `plan:internal/shedverbs#statusStopWayForward` -> `func statusStopWayForward(spec *Spec, cause string, readFailed bool) string`

**Edit:**
- `internal/shedverbs#runLoop`
- `internal/shedverbs#childResult`
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#Spec`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#interruptedError`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Uses:**
- `internal/shedverbs#newLoopFixture`

**Intent:**
D5: the loop's top-of-iteration status read tells an absent file from an unreadable one, re-reads a read error once, and points its stop at the status path and its own log.

`LoopSpec` gains the unexported field `readStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`; nil reads through `state.ReadJSONNoMkdir[shedengine.Status]`, and a test sets it to fail a read on cue.

`readLoopStatus` (in `internal/shedverbs/loop.go`) reads `spec.StatusPath` through that reader once.
It returns an empty cause for a read file, `status-missing` for an absent one, and `status-unreadable: <error>` for any error; `readFailed` is true for a read-lock or raw read error and false for a decode error, told apart by `errors.Is(err, state.ErrDecode)`.
At either non-empty cause it writes one `logger.Warn` naming the status path and, when unreadable, the error.

`statusStopWayForward` returns the way-forward clause of a stop at such a cause, each naming the status path and the loop log `spec.Loop.LogPath`:
- `status-missing`: the recipe's reseed, `spec.MissingStatusWayForward` with its leading `way forward: ` trimmed (loom's names `lyx loom start`, batten's `lyx batten run <slug>`), or, when that is empty, `run "lyx shed seed <run-id> --recipe <name>"`;
- `status-unreadable` with `readFailed`: re-run `lyx shed step <run-id> --until-stop` once the status file reads again;
- `status-unreadable` from a decode error: re-run `lyx shed step <run-id> --until-stop` with a lyx binary that decodes the file, as after a version skew where an older loop binary cannot decode a file a current one reads, or else report the stop to the run's parent with the status path and the loop log.

`childResult` gains the field `statusWayForward string`, set only for a stop on the status read.

`runLoop` replaces its `readStatusFile` call at the top of each iteration with `readLoopStatus`:
- an empty cause runs the child as today;
- `readFailed` with no re-step spent sets `restepped`, records `obj.Restep` with the last known producer, the cause as its error and transient `status-read`, logs the re-step and continues, so a read that clears lets the loop step on under the same single re-step a transient child error uses;
- otherwise the result is `childResult{traceID, cause, statusWayForward}` with no child run.
At an interrupted stop whose `statusWayForward` is set, the error text is `interruptedError`'s message plus `; way forward: ` and that clause, the trace file handed to `Hooks.AfterInterrupt` and to `interruptedRecord` is `spec.Loop.LogPath`, as the `loop-exited` stop does, and `shedengine.WriteFailedStop` is not called, so the status file stays as found.
Every other stop is unchanged.
The cause keeps the `<class>: <detail>` form `convergence-plan` classes halt reasons by.
No re-step on a decode error: `internal/state` writes the file by atomic rename under the status lock, so a file that fails to decode stays undecodable to that binary.
The bound, a stated departure from `PATTERN-failure-stop-visible`: at these causes the status file is not made non-running, since `WriteFailedStop` cannot write over an absent or undecodable file and overwriting it would destroy the evidence.

Doc comments: `interruptedError`'s, which says the envelope, the status file's error and the refusal row all name its way forward, says a stop on the status read takes `statusStopWayForward`'s clause instead and writes no status error; `Spec`'s `MissingStatusWayForward` says the loop's `status-missing` stop names it too, and `Hooks`' `AfterInterrupt` says a stop on the status read hands the loop log as the trace file.

`TestLoop_InterruptedChildFillsTheErrorForm`'s subtest `status file vanishes between steps` becomes the table subtest `status read at the top of an iteration`, built on `newLoopFixture`, whose first child continues to `B` and then breaks the status as each row says.
Each stopping row also asserts, through `logcapture.Capture` from `internal/testkit/logcapture`, that the captured output holds the status-read Warn naming the status path and no `could not write the failed stop` line, which pins the skipped `WriteFailedStop`; every stopping row's error holds the status path and the loop log path.
- absent, with `fx.spec.MissingStatusWayForward` set to `way forward: run "lyx loom start" first to bootstrap this task`: exit 1, kind interrupted, error holding `step B interrupted (status-missing)`, `lyx loom start`, the status path and the loop log path; one child started; `trace_file` and the `AfterInterrupt` trace argument equal to `LogPath`; the status file still absent;
- absent with no told clause: the error holds `lyx shed seed run-1 --recipe <name>`;
- the file rewritten to `{`: error holding `step B interrupted (status-unreadable: `, `a lyx binary that decodes` and `report the stop to the run's parent`; one child started; the file still exactly `{`;
- `fx.spec.Loop.readStatus` failing with a plain error on its first call after the child and then reading through: a second child runs and halts, the envelope is ok and `loop.restep` names transient `status-read`;
- `readStatus` failing on every call after the child: error holding `status-unreadable: ` and `once the status file reads again`; one child started.

`contracts/specs/refusal-spec.md`'s shed section changes in two rows.
`step interrupted` drops "or the status file vanished" and adds: a status read at the top of an iteration that fails with a read-lock or raw read error, and again on its one re-read, stops with cause `status-unreadable: <error>`, leaves the status file as found rather than writing it `failed`, names the loop log as the trace file, and its way forward is re-running `lyx shed step <run-id> --until-stop` once the file reads, with the status path and loop log named.
A new row `loop status missing or undecodable`, right after it and a correctness halt like `status file missing`, since neither cause clears by itself or by a retry: the read finds no status file (cause `status-missing`) or one that does not decode (cause `status-unreadable: <error>`); no child runs, the stop is `interrupted`, the status file is left as found rather than written `failed`, and the loop log is the trace file; its way forward is the recipe's reseed as the `status file missing` row gives it at `status-missing`, and at `status-unreadable` a re-step with a binary that decodes the file, or else reporting the stop to the run's parent with the status path and loop log.

`pattern/PATTERN-failure-stop-visible.md` gains a bullet: a loop stop on an absent or undecodable status file, or one a read error kept unreadable through its re-read, leaves the file as found, so the status is not made non-running there; the halt note names the status path and the loop log instead.
`PATTERN.md`'s one-liner is left as it is: the overview sits at its headroom line, so the departure lives in the background alone.

Docs: `internal/shedverbs/doc.go`'s stop paragraph replaces "including a status file that vanished; both write the status file failed" with the two status causes, the single re-read, the loop log as trace file and the file left as found, and lists `status-read` beside the re-step it reports.

The commit body closes #581.

**ImpactSummary:** a loop stop on an absent or unreadable status file now names its cause, its reseed or re-step and the loop log, and leaves the file untouched; every child-step stop is unchanged.

**Commit:** 4: shed: tell an absent loop status from an unreadable one and name the loop log
