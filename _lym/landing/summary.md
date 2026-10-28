# Retry loop child starts on ETXTBSY and install lym by rename

A loop child start refused with `text file busy` (ETXTBSY) no longer ends the run `failed` at once.
The binary is no longer rewritten in place by deploy, and a start that still fails explains itself in the halt note.

Closes #893

## What changes

- `internal/shedverbs`: `startChild` starts the loop's child again when the start is refused with ETXTBSY, five tries in all.
  The first retry waits 100 ms and each later wait doubles it.
  The waits go through `LoopSpec.Sleep`, and each retry logs at Warn.
  Any other start error fails at once, as before.
- `internal/proc`: `IsSpawnBusy` is the one ETXTBSY predicate, beside the existing EAGAIN one.
- `tools/deploy`: `installBinary` builds `lym` into a temporary file beside the target and renames it over the target.
  `update-plugins.sh`, `deploy-dev` and the Land row's `deploy` command all run this tool, so all three install atomically.
  A failed build or rename removes the temporary file and leaves the target as it was.
- A start that still fails ends the stop with the cause `start-failed after <n> tries: <error>`, where `<n>` is the tries made.
  For ETXTBSY the cause adds that the executable was open for writing, as while a deploy replaces it in place.
- An interrupted stop whose child left no trace file and no stderr now records the loop process's own trace file, which holds the start warnings.
  `LoopSpec.LoopTraceFile` carries it, wired to `logger.TraceFile` in `internal/loomcli/looparm.go` and `internal/battencli/arm.go`.

## Design decisions

- ETXTBSY is refused before the child runs, so a retry never runs a step twice.
  The bound of about 1.5 s covers a binary copy while a binary held open for writing still fails.
- `go build -o` copies into an existing target in place when its work directory is on another filesystem, and that window is what a spawning process sees as ETXTBSY.
  The temporary file sits in the install directory rather than the lym temp root because a rename across filesystems is not atomic.
- No new anomaly kind or transient class is added for a failed start; it stays an interrupted stop.
- Other `lym` spawn sites are not changed.
- On Windows a rename over a running `lym.exe` fails, as `go build -o` over it did before.

## Check by hand

Not run: a real deploy racing a loop child start.
A deploy that holds the binary open for writing longer than the retry bound still ends the run `failed`, now with a cause that names it.

Check these points in the diff:

- `internal/shedverbs/loop.go`: the `start-failed after <n> tries` cause is built for every start error, with the tries actually made, including a backoff that the context cancels early.
- `internal/shedverbs/loop.go`: the loop-trace fallback applies to any interrupted stop with an empty trace file, not only to a child that never started.
- `internal/shedverbs/loopchild.go`: a cancelled backoff reports the tries made, not all five.
- `internal/shedverbs/loop_test.go`: the fallback's empty-path cases, `LoopTraceFile` nil and returning an empty string, keep `trace_file` empty without a panic.
- `tools/deploy/main_test.go`: the rename-failure branch of `installBinary` removes the temporary file and leaves the target unchanged.
