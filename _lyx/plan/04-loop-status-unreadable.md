# Card 4 — loop-status-unreadable

**Create:**
- `plan:internal/shedverbs#readLoopStatus` -> `func readLoopStatus(spec *Spec) (shedengine.Status, bool, error)`
- `plan:internal/shedverbs#statusStopWayForward` -> `func statusStopWayForward(spec *Spec, missing, decode bool) string`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#childResult`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#Spec`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#interruptedError`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs#TestLoop_TransientRestepOnce`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `PATTERN.md`
- `pattern/PATTERN-failure-stop-visible.md`

**Intent:** D5: the loop tells an absent `status.json` from an unreadable one, and its stop points at its own log, so a halt note is diagnosable (#581).

`LoopSpec` gains a field `ReadStatus func() (shedengine.Status, bool, error)`, the loop's status read at the top of each iteration; nil selects the production read.
`readLoopStatus` calls it when set, and otherwise `state.ReadJSONNoMkdir[shedengine.Status](spec.StatusPath, spec.StatusLockPath)`, returning the status, whether the file was found, and the read error.
Only the iteration read in `runLoop` goes through it; the `first` and `after` reads and every other `readStatusAt` caller stay as they are.

`childResult` gains a field `wayForward string`, set only at a status stop, and `runLoop`'s iteration read becomes:
- found: as today.
- not found, no error: `childResult{traceID, cause: "status-missing", wayForward: statusStopWayForward(spec, true, false)}`.
- an error wrapping `state.ErrDecode`: `cause: "status-unreadable: " + err.Error()`, `wayForward: statusStopWayForward(spec, false, true)`; no re-step, since `internal/state` writes the file by atomic rename under the status lock, so it stays undecodable to this binary.
- any other error (a read-lock or raw read error) with `restepped` false: set `restepped`, record `obj.Restep = &loopRestep{Producer: lastKnown, Error: err.Error()}` with an empty transient class, `logger.Warn` it, and `continue`, which re-reads at the next iteration's top; with `restepped` already true: `cause: "status-unreadable: " + err.Error()`, `wayForward: statusStopWayForward(spec, false, false)`.
At either status cause the loop writes a `logger.Warn` naming `spec.StatusPath` and, for `status-unreadable`, the error.

In the `LoopStopInterrupted` branch, a result carrying `wayForward` uses it instead of `interruptedError`'s way forward (the message stays `interruptedError`'s, so the cause keeps its `<class>: <detail>` form after `interrupted (`), and its trace file, handed to `Hooks.AfterInterrupt` and to `interruptedRecord`, is `spec.Loop.LogPath`, as `loopExitStop` already does for `loop-exited`.
Such a stop skips `shedengine.WriteFailedStop`, so the status file is left as found: an absent file stays absent and an undecodable one keeps its bytes; that is D5's bound and its stated departure from `PATTERN-failure-stop-visible`.

`statusStopWayForward` names the status path and the loop log and, by cause:
- missing: the recipe's own reseed, which is `spec.MissingStatusWayForward` with its leading `way forward: ` removed (`lyx loom start` for loom, `lyx batten run <slug>` for batten, both already told by their arming modules), or `lyx shed seed <run-id> --recipe <name>` when none is told;
- decode: re-run `lyx shed step <run-id> --until-stop` with a `lyx` binary that decodes the file, as after a version skew, or otherwise report the stop to the run's parent with the status path and the loop log, as refusal-spec's `arbiter: status or envelope unreadable` row does;
- read error that failed its re-read: re-run `lyx shed step <run-id> --until-stop`.
No branch names a command an agent's settings deny.

`Hooks.AfterInterrupt`'s doc comment says that at a status stop its trace file is the loop log.
`Spec.MissingStatusWayForward`'s field comment, which says the clause is passed to goto, adds that the loop's `status-missing` way forward reads it too.
`interruptedError`'s doc comment, which says the envelope, the status file's error and the refusal row all name its way forward, says that a status stop replaces that way forward with `statusStopWayForward`'s and writes no status file; its code does not change.

Tests; the file keeps no `t.Parallel`, so a subtest may capture the process-global logger with `logger.SetOutput` into a buffer and restore it with `t.Cleanup`.
In `TestLoop_InterruptedChildFillsTheErrorForm`:
- "status file vanishes between steps" also sets `fx.spec.MissingStatusWayForward` to `way forward: run "lyx loom start" first`, records the `AfterInterrupt` arguments, captures the logger, and asserts the error holds `interrupted (status-missing)`, `lyx loom start`, the status path and the loop log path, the envelope's `trace_file` and the hook's trace file are `fx.spec.Loop.LogPath`, one child ran, the status file is still absent, and a `level=WARN` line names the status path.
- New subtest "a vanished status file with no told reseed names lyx shed seed": the same script with `MissingStatusWayForward` left empty; it asserts the error holds `interrupted (status-missing)` and `lyx shed seed run-1`, the fallback branch.
- New subtest "an undecodable status file stops without a child": the child script runs `continuing("B", 1)` and then overwrites the status file with `{`; it captures the logger and asserts one child, an error holding `interrupted (status-unreadable: ` and the way forward naming a binary that decodes the file and the run's parent, `trace_file` the loop log, the status file's bytes still `{`, and a `level=WARN` line naming the status path and the decode error.
- New subtest "a status read error that fails its re-read stops": `ReadStatus` fails with `state.ErrRead` from its second call on; it asserts one child, an error holding `interrupted (status-unreadable: ` and `re-run lyx shed step run-1 --until-stop`, `trace_file` the loop log, and the status file still reads `running`, which only the explicit skip of `WriteFailedStop` keeps.
In `TestLoop_TransientRestepOnce`, whose table pins a re-step's `loop.restep`, step count and final stop:
- The table gains a `readStatus` field installed as `fx.spec.Loop.ReadStatus` when set, and a `wantTransient` field that, when set, replaces the `class` the `loop.restep` assertion expects, with a sentinel or a `*string` so the row can expect the empty class.
- New row "a status read error that clears on the re-read steps on": `readStatus` fails once with an error wrapping `state.ErrRead` on its second call (the iteration after child 1, scripted `continuing("B", 1)`), then reads the real file; child 2 halts `done`; it wants stop `halted`, two steps, `loop.restep` naming producer `B`, the read error and an empty transient class, and status `done`.

Docs: `internal/shedverbs/doc.go`'s stop paragraph states the two status causes, that they run no child, that their trace file is the loop log and that they leave the status file as found.
`contracts/specs/refusal-spec.md`'s shed section: the `step interrupted` row drops "or the status file vanished" and adds `status-unreadable: <error>` from a status read error that failed its one re-read, at which the status file is left as found rather than written `failed`, its way forward re-running `lyx shed step <run-id> --until-stop`; a new row `loop status missing or undecodable`, a correctness halt like the `status file missing` row, triggers on `status-missing` and on `status-unreadable` from a decode error, states that the status file is left as found rather than written `failed` and that the loop log is the trace file, and gives the way forward by cause as `statusStopWayForward` does.
`PATTERN.md`'s `PATTERN-failure-stop-visible` line, the inlined form spawned agents read, names the exception:

```
- `PATTERN-failure-stop-visible` — An `--until-stop` failure as sole lock holder: status is non-running before the waiter returns, unless the status file cannot be read. (test) — [background](pattern/PATTERN-failure-stop-visible.md)
```

`pattern/PATTERN-failure-stop-visible.md` gains a bullet: a stop on an absent or undecodable status file, or one whose read fails twice, leaves the file as found, since `shedengine.WriteFailedStop` cannot write over it and overwriting would destroy the evidence; the way forward's re-step resumes it, not `lyx loom resume`.

**ImpactSummary:** Only `runLoop`'s iteration read and its interrupted branch change; `loopExitStop`'s `loop-exited` path, the `first`/`after` reads and every recipe's arming are untouched, and `LoopSpec.ReadStatus` stays nil in production.

**Commit:** `4: Loop stop tells a missing status file from an unreadable one and names its log (#581)`
