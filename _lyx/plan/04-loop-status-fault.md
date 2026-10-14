# Card 4 — loop-status-fault

**Rename:**
- `internal/shedengine#missingStatusClause` -> `plan:internal/shedengine#MissingStatusClause`

**Create:**
- `plan:internal/shedverbs#readLoopStatus` -> `func readLoopStatus(spec *Spec) (shedengine.Status, bool, error)`
- `plan:internal/shedverbs#statusFaultWayForward` -> `func statusFaultWayForward(spec *Spec, readErr error) string`

**Edit:**
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#childResult`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#TestLoop_InterruptedChildFillsTheErrorForm`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-failure-stop-visible.md`
- `PATTERN.md`

**Intent:** The `--until-stop` loop tells an absent status file from an unreadable one at the top of an iteration and points its stop at its own log (D5), so a halt note on such a stop says what failed and where to read more (#581).

Today `readStatusFile` reports false for an absent file and for any read or decode error alike, the loop stops with cause `status-missing`, no child runs, so the interrupted record and `Hooks.AfterInterrupt` get no trace file, and the way forward is the generic re-run.

- `shedengine`'s `missingStatusClause` is exported as `MissingStatusClause`, unchanged in body, so the loop can name the recipe's own reseed with the same generic fallback `Step` and `Goto` use; `Goto`, `Relaunch` and `Shed.stepLocked` call it by its new name.
  It is a symbol rename inside `run.go`: no file moves, so the Rename mechanic's `git mv` step has nothing to move.
  D5 requires the loop to name the reseed "as refusal-spec's `status file missing` row gives it", which is this clause; exporting it is the only way the loop can name the same text without a second copy of the generic fallback in shedverbs, so the rename is in D5's scope.
- `LoopSpec` gains the field `ReadStatus func(statusPath, statusLockPath string) (shedengine.Status, bool, error)`, documented like `Now` and `Sleep`: nil selects `state.ReadJSONNoMkdir[shedengine.Status]`; no arming module sets it.
- `readLoopStatus` reads `spec.StatusPath` under `spec.StatusLockPath` through that field and returns the status, whether the file was found, and the read's error.
  It is the top-of-iteration read alone: `runLoop`'s `first` read before the loop and `after` read at the stop, and `ReportLoopArmError`, keep reading through `readStatusFile` and `readStatusAt`, which never consult `ReadStatus`.
  The field is a test seam, the same kind as `Now` and `Sleep`, and lets a test inject a read error that clears or persists; it skips nothing in production.
- `childResult` gains a `statusWayForward string` field, set only for a stop on the status read, which marks the stop as a status fault and carries its way forward.
- In `runLoop`'s iteration the read at the top goes through `readLoopStatus`:
  - found: the child runs as today.
  - absent (no error, not found): cause `status-missing`.
  - an error wrapping `state.ErrDecode`: cause `status-unreadable: <error>`, with no re-read, since the only writer, `internal/state`, writes by atomic rename under the status lock, so the file stays undecodable to this binary.
  - any other error (the read lock or the raw read): transient. When the loop has not re-stepped since its last success it sets `restepped`, records `obj.Restep` with the last known producer and the error, logs the re-read and re-reads at once; when the re-read fails too, it stops with cause `status-unreadable: <error>`.
  - At either cause the loop writes a `logger.Warn` naming the status path and, when unreadable, the error.
- `statusFaultWayForward` builds the way forward from the read's error, and each form names the status path and the loop log `spec.Loop.LogPath`:
  a nil error (the file is absent) names the recipe's reseed, `MissingStatusClause(spec.MissingStatusWayForward)` with its leading `way forward: ` removed, which is `lyx loom start` for loom, `lyx batten run <slug>` for batten and `lyx shed seed` otherwise;
  a decode error names re-stepping with a binary that decodes the file, `lyx shed step <run-id> --until-stop`, as after a version skew, and otherwise reporting the stop to the run's parent with the status path and the loop log;
  any other error names re-running `lyx shed step <run-id> --until-stop`.
- At the interrupted stop, a status fault takes `res.statusWayForward` in place of `interruptedError`'s way forward, keeps `interruptedError`'s message with the cause, and uses `spec.Loop.LogPath` as the trace file handed to `Hooks.AfterInterrupt` and to `interruptedRecord`, as the waiter's `loop-exited` stop already does.
  It skips `shedengine.WriteFailedStop`, so the status file is left as found: an absent file stays absent, an undecodable one keeps its bytes, and one whose read failed is not overwritten.
  `runLoop`'s doc comment, "an error or interrupted stop on a run still reading running writes it failed", gains the exception for these two causes.

Bound: at these causes the status is not made non-running, a stated departure from `PATTERN-failure-stop-visible` for the case where no status can be read; overwriting it would destroy the evidence.
The halt note's history stays empty, since loom's hook re-reads the file.
A file that later reads `running` is resumed by the way forward's re-step, not by `lyx loom resume`, and `lyx loom arbiter` refuses either stop under its `status or envelope unreadable` row.
The transient re-read shares the loop's single re-step: a read error right after a child's transient re-step stops at once.
The cause keeps the `<class>: <detail>` form `start-failed: <error>` uses, so `convergence-plan` classes it by its leading clause.

Tests: `TestLoop_InterruptedChildFillsTheErrorForm`'s "status file vanishes between steps" subtest becomes a table subtest "the status file cannot be read at the top of an iteration".
Every row's first child is `continuing("B", 1)` followed by the row's fault, sets `fx.spec.Hooks.AfterInterrupt` to record the cause and trace file it is handed, and asserts exit 1, kind `interrupted`, the error's cause and way-forward substrings, the status path and the loop log path in the error, the interrupted record's `trace_file` and the hook's trace file equal to `fx.spec.Loop.LogPath`, the number of children started, and the status file as the row left it.
Rows:

- the file removed, with `fx.spec.MissingStatusWayForward` set to `way forward: run "lyx loom start" first`: cause `status-missing`, way forward holds `lyx loom start`, one child, the file still absent;
- the file removed with no told clause: way forward holds `lyx shed seed`;
- the file overwritten with `{`: cause `status-unreadable: ` with `state: decode failed`, way forward holds "binary that decodes" and "parent", one child, no re-read (`fx.spec.Loop.ReadStatus` is a counting fake that delegates, called exactly twice: once per iteration), the bytes still `{`;
- `fx.spec.Loop.ReadStatus` a fake that delegates on its first call and fails every later call with a plain error, while the file on disk stays a readable `running` status: cause `status-unreadable: ` with that error, way forward holds `lyx shed step run-1 --until-stop`, one child, the fake called three times (the first iteration's read, the failing read and its one re-read), and the file on disk still reads `running`, which fails if the failed write is not skipped, since `WriteFailedStop` could write over this readable file;
- `fx.spec.Loop.ReadStatus` a fake that fails its second call with a plain error and delegates otherwise, scripts `continuing("B", 1)` then `halting(shedengine.StateDone, "", 2)`: the loop steps on, two children, stop `halted`, and the envelope's `loop.restep` names the read error; this row skips the interrupted assertions.

Docs in this commit:

- `contracts/specs/refusal-spec.md`'s `## shed` table: the `step interrupted` row drops "or the status file vanished" and gains the transient case `status-unreadable` from a read error that failed its re-read, stating that at that cause the status file is left as found rather than written `failed`, the trace file is the loop log, and the way forward is the re-step.
  A new row `loop status missing or undecodable`, class correctness halt like `status file missing`, names `status-missing` and `status-unreadable` from a decode error, that the status file is left as found, and each cause's way forward above.
- `internal/shedverbs/doc.go` states the two causes, the single re-read, the loop log as trace file and the skipped failed write.
- `PATTERN.md`'s `PATTERN-failure-stop-visible` line becomes, verbatim:

```markdown
- `PATTERN-failure-stop-visible` — An `--until-stop` failure as sole lock holder: status is non-running before the waiter returns, unless no status can be read. (test) — [background](pattern/PATTERN-failure-stop-visible.md)
```

- `pattern/PATTERN-failure-stop-visible.md` gains a bullet: a stop on an absent or unreadable status file (`status-missing`, `status-unreadable`) writes nothing, since there is no status to make non-running and overwriting it would destroy the evidence; its way forward names the reseed or the re-step instead.

**ImpactSummary:** Changes only the loop's top-of-iteration status read and the stop it reports there, plus one same-package export in shedengine with its three callers; every stop on a readable status file behaves as before.

**Commit:** `4: shed loop tells an absent status file from an unreadable one`
