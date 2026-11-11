```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens for the shed loop-keeper diff.

**Per function and branch (case I ran, then correct or not)**

- `keepLoop`, loop exits 3 with unterminated output: it writes the mark on its own line, `process N exited: exit status 3`, and returns 1. Correct.
- `keepLoop`, loop SIGKILLed: `cmd.Wait` returns `signal: killed`, the mark is written and the keeper returns 1. Correct.
- `keepLoop`, `cmd.Start` fails (missing executable): it writes the `could not start: …` mark, skips `OnStart` and returns 1. Correct.
- `keepLoop`, keeper receives SIGTERM or SIGHUP: the goroutine logs it and `cmd.Wait` carries on, so the keeper lives as long as the loop. `Signals` returning a nil channel in tests is harmless. Correct.
- `keepLoop`, log directory removed: the mark is written through the inherited descriptor, so the log is not recreated. Correct. A write failure is only logged, which is fine.
- `keepLoop` with `OnStart`: it runs after a successful start, before `Wait`, once. Correct. Whether registering the keeper and the loop under the same slug and role collides depends on `runproc.Register` keying, which I did not read. I found no defect, and I name it as the one unverified point.
- `stepCmd` keeper branch: `childArgv` carries no flags (`step.go:91-99`), so the keeper's `loopArgv` holds only `--until-stop` and `--loop-detached`, and the loop never re-spawns a keeper. Correct.
- `stepCmd` with both flags set: the keeper wins and `validLoopID` guards both. Correct.
- `DetachedLoopID` with neither flag set: it returns "". Correct.
- `spawnLoop` reap mark: it writes `keeper process N exited: …` and closes `exited` after the mark. Correct. Since the keeper outlives the loop, the waiter's `spawnExited` now means the loop is gone and its mark is written. The start handshake still holds.
- `loopExitEvidence`: the new marks sit after the last `starting` mark and are listed in log order, including the `keeper process` mark. Correct.
- `deadLoopStop` with a child in flight and the status moved: it names the trace, then `, and the status file now reads B`. Correct.
- `deadLoopStop` with the child cleared: no in-flight clause, and the trace file stays `loop.log`. Correct.
- `deadLoopStop` with an unreadable status file: `CurrentProducer` is "", so no "now reads" clause. Correct.
- `deadLoopStop` for the teardown mark: `hasChild` is false and the mark detail is unchanged. Correct.
- `loopExitStop`, trace file: `rec.TraceID != "" && rec.hasChild()` selects the first trace file even after the status moved. Correct.
- `loopExitStop`, `loop.stderr_path` and `loop.trace_copy`: both are now set whenever a child is recorded. Correct. `StatusMoved` keeps its meaning.
- `loopExitedEarlyStop`: the record has no child, so it uses the loop log. Correct.
- `runChild` clearing the child record after `Wait`: a loop that dies after the child exits but before the clear would name a finished child. The window is tiny and harmless.

**Findings**

1. LOW, `design`: `loopwait.go:96-99` and `loopwait.go:354-359`, with `loopkeeper.go:57-62`.
   - A waiter that did not spawn the keeper treats a free loop lock plus a pid record as a dead loop at once.
   - That covers a waiter following a loop another driver started, or one re-run after a handover, where `follow.spawned` is nil.
   - The loop releases its lock when it dies, and the keeper writes its exit mark only after `Wait` returns. In that window the waiter can report `loop exit: not recorded`, which the entry is meant to remove.
   - Only a waiter that spawned the keeper is ordered after the mark.
   - The integration test avoids the race by waiting for the mark first (`loop_unix_integration_test.go:74-79`).
   - A fix would be a short re-read of the log for the loop's exit mark before reporting, or ordering the keeper's mark before the lock release.

2. NIT, `design`: `loopwait.go:518-520`.
   - The "and the status file now reads X" clause also fires when X equals the in-flight producer and only the history length moved.
   - The note then reads "step A was in flight … and the status file now reads A", which is misleading but not wrong.

**Tool calls**
1. `grep` in `internal/shedverbs` for `childArgv`, and in `internal/runproc` for `Register`, `SelfRecord` and `End`: the manifest lacked `childArgv`'s body. I needed it to confirm the keeper flag does not reach the child or loop argv.
