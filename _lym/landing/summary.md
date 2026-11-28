# Show each tier's slot wait apart from its run time in `lym gate test`

Timing a `lym gate test` call from outside also times the wait for gate slots.
Each tier's summary line already printed the run's own time as `wall`, but not the slot wait.

The stderr summary line now adds `wait` beside the unchanged `wall`:

```text
gate test: tier<N> <scope> wait <s>s discovered <n> passed <n> failed <n> skipped <n> wall <s>s exit <code> log <path>
```

`wait` is the time the call waited for the tier's slots, and `wall` is the run's own time without that wait.
The verb's help and the `gatecli` package doc say so.

Closes #284

## Design decisions

- The wait ends where the slots are held, or are found to be unneeded, not where the tier's command starts.
  The shared build runs inside the held slots, so it is no part of `wait`; its time is on the build's own line.
  The `lym` prebuild of a `--copies` run is in neither figure, while a run without copies counts it in `wall`.
- The run record gains `slots_held`, and `RunRecord.SlotWaitSeconds` ends the wait there.
  A record without the field ends its wait at `start`.
- `lym gate report` takes its wait totals from the same method,
  so for new records they no longer include the shared build or the `--copies` prebuild.
- A rerun's line shows the wait the call spent on the tier's slots.
  The rerun's own record still holds no wait, so the report does not count it twice.
- `wait` stands before `discovered`, where the existing summary-line patterns accept any text.
  The patterns are unchanged, so a log and a reader built from different versions still agree.
- To keep the verb's long help under its byte cap, its `--copies` paragraph is dropped.
  The flag's help and the package doc still describe the flag.

## To check by hand

- The printed wait on the real tier paths, set in `internal/gatecli/run.go`.
  `internal/gatecli/slotwait_test.go` covers a slotted run, a carried tier, a shared-build tier, a rerun
  and a `--copies` run with a stand-in clock.
  No row waits for a busy slot.
- A rerun tier's summary line must show the slot wait and not `0.0s`, in `internal/gatecli/run.go#report`.
- The summary-line patterns in `internal/gateslot/runlog.go` must read a line both with and without the `wait` field,
  since the binary that wrote a log need not be the one reading it.

## Merge danger

**Door:** one-way
**Blast radius:** gate

The door is one-way only in that run records are a format other binaries read, and they gain `slots_held`.
An older binary ignores the field.
