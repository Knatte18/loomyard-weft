# Card 4 — The loop tells an absent status from an unreadable one and points at its own log

**Create:**
- `plan:internal/shedverbs#readLoopStatus` -> `func readLoopStatus(spec *Spec) (shedengine.Status, bool, error)`
- `plan:internal/shedverbs#statusReadStop` -> `func statusReadStop(spec *Spec, traceID string, readErr error) childResult`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#childResult`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Uses:**
- `internal/shedverbs#interruptedError`

**Intent:**
D5 (#581): `status-missing` is set when the top-of-iteration read reports false, which today covers an absent file and every read or decode error alike, with no history entry, no trace file and a generic way forward, so the halt note says neither what failed nor where to read more.

`LoopSpec` gains a field `ReadStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`: the read at the top of each loop iteration, nil selecting `state.ReadJSONNoMkdir[shedengine.Status]`.
It is a test seam beside `Now`, `Sleep` and `Watch`, and no arming module sets it.
The decision record names no seam; it is an implementation choice so a test can reach the acceptance row where a read error clears on the re-read, which no real file can produce between two reads.
`readLoopStatus`, in `loop.go`, calls it, or the default, over `spec.StatusPath` and `spec.StatusLockPath`, and returns its three results.
The read before the loop and every other status read keep `readStatusFile` and `readStatusAt`.

`childResult` gains a field `wayForward`, the way forward a stop that ran no child names in place of `interruptedError`'s generic one; empty for every other result.

`statusReadStop`, in `loop.go`, builds the result of a top-of-iteration read that found no status, with the trace id, cause and way forward, and writes one `logger.Warn` naming the status path, and the error when there is one.
`run` below is `spec.RunID`, `status` is `spec.StatusPath` and `log` is `spec.Loop.LogPath`:
- `readErr` nil, the file absent: cause `status-missing`; way forward `<reseed>, since the status file <status> is absent; the loop log is <log>`, where `<reseed>` is `spec.MissingStatusWayForward` with its leading `way forward: ` removed (loom's names `lyx loom start`, batten's `lyx batten run <slug>`), and, when that is empty, `seed the run through its recipe's bootstrap verb, or "lyx shed seed" for a recipe without one`.
- `readErr` wrapping `state.ErrDecode`: cause `status-unreadable: <readErr>`; way forward `re-run lyx shed step <run> --until-stop with a lyx binary that decodes the status file <status>, as after a version skew; otherwise report the stop to the run's parent with the status file <status> and the loop log <log>`.
  No re-step is made: `internal/state` writes the file by atomic rename under the status lock, so a file that fails to decode stays undecodable to this binary.
- any other `readErr`, a read-lock or raw read error that failed its re-read: cause `status-unreadable: <readErr>`; way forward `re-run lyx shed step <run> --until-stop; the status file <status> could not be read and the loop log is <log>`.

`runLoop` reads through `readLoopStatus` at the top of each iteration.
A found status runs the child as today.
A read error that is neither absent nor `state.ErrDecode` uses the loop's existing single re-step: when `restepped` is false it sets it, logs the retry and continues, so the next iteration re-reads; a re-read that succeeds steps on, and one that fails, or a read error with the re-step already spent, stops.
Every other no-status read stops at once with `statusReadStop`'s result.
At the interrupted stop, a result with a `wayForward` uses it in place of `interruptedError`'s way forward, keeps `interruptedError`'s message (`shedverbs: step <producer> interrupted (<cause>)`), and takes `spec.Loop.LogPath` as the trace file handed to `Hooks.AfterInterrupt` and to the interrupted record, as the waiter's `loop-exited` stop already does.
Such a stop skips `shedengine.WriteFailedStop`, so the status file is left as found: absent, undecodable, or as the failed reads left it.
This is a stated departure from `PATTERN-failure-stop-visible` for a stop that reads no status: `WriteFailedStop` cannot write over an absent or undecodable file, and overwriting a file the loop could not read would destroy the evidence.
The causes keep the `<class>: <detail>` form, so `convergence-plan` classes the halt reason by its leading clause.
Loom's `loomAfterInterrupt` needs no change: it writes the halt note from the cause and the trace file it is handed, and its history stays empty since it re-reads the file.

`runLoop`'s doc comment says that an interrupted stop on a status the loop could not read writes nothing, in place of its claim that every interrupted stop on a run still reading running writes it failed.

`TestLoop_InterruptedChildFillsTheErrorForm`'s `status file vanishes between steps` subtest becomes a table subtest `a status the loop cannot read stops with the cause and the loop log`.
In each row the first child continues to `B` with history 1 and then acts on the status file, unless the row says otherwise, and `Hooks.AfterInterrupt` records its arguments.
Every row that stops also asserts that `trace_file` and the hook's trace file equal `LogPath`, and that the `logger.Warn` captured through `internal/testkit/logcapture`'s `Capture` names the status path.
The rows do not run in parallel, since `Capture` redirects the process-global logger.
Rows:
- the child removes the status file, with `spec.MissingStatusWayForward` told as loom's `way forward: run "lyx loom start" first to bootstrap this task`: exit 1, kind interrupted, the exact error `shedverbs: step B interrupted (status-missing); way forward: run "lyx loom start" first to bootstrap this task, since the status file <status> is absent; the loop log is <log>`, `trace_file` and the hook's trace file equal to `LogPath`, the hook's cause `status-missing`, one child, and the status file still absent.
- the same with no told clause: the error carries the generic seed clause.
- the child writes `{` to the status file: the hook's cause starts with `status-unreadable: state: decode failed`, the error carries `with a lyx binary that decodes the status file` and `report the stop to the run's parent`, one child, and the file's bytes are still `{`.
- `ReadStatus` fails once with an error wrapping `state.ErrRead` on its second call, then delegates: the loop steps on to a second child that halts done, and the stop is halted.
- `ReadStatus` fails with an error wrapping `state.ErrRead` from its second call on: the hook's cause starts with `status-unreadable: `, the error carries `re-run lyx shed step run-1 --until-stop; the status file`, one child, and the status file still reads `running` at `B`, never written `failed`.
- the first child fails with a transient producer error, which spends the re-step, and `ReadStatus` fails once with an error wrapping `state.ErrRead` on its second call: the loop stops at once with a cause starting `status-unreadable: `, after one child.

`contracts/specs/refusal-spec.md`'s shed section changes two rows.
The `step interrupted` row, transient, drops `or the status file vanished` and adds the cause `status-unreadable: <error>` from a read error that failed its re-read, at which the status file is left as found rather than written `failed`, and whose way forward is to re-run `lyx shed step <run-id> --until-stop`.
A new row `loop status missing or undecodable`, a correctness halt like the `status file missing` row because neither cause clears by itself or by a retry, names `status-missing` and the decode-error `status-unreadable: <error>`, states that the status file is left as found, that the stop's trace file is the loop log, and gives each cause's way forward above.
`pattern/PATTERN-failure-stop-visible.md` gains a bullet stating that departure: a stop on an absent or undecodable status file, or on one the loop could not read twice, writes nothing, and its way forward names the re-step or the reseed.
`internal/shedverbs/doc.go`'s stop paragraph replaces `including a status file that vanished` with the two causes, the loop log as their trace file, the single re-read and the file left as found.
Its later sentence that `Spec.Loop.OnExit` runs after the stop is final and a failed stop is written says the failed stop is written only where one applies.

**ImpactSummary:** the loop's interrupted stop on an unreadable or absent status now names its cause, the status path and the loop log, and no longer writes the status file; every stop with a readable status is unchanged.

**Commit:** 4: tell an absent loop status from an unreadable one in the stop
