# loom run halted failed at Merge-In with status-missing, no history and no trace file

`merge-in-status-missing` · Note · bug

Imported from [issue #581](https://github.com/Knatte18/loomyard/issues/581).

`lyx loom` halted a run as `failed` at the Merge-In producer.
The halt note shows reason `status-missing`, anomaly `producer-hard-failure`, `history_entries: 0` and an empty `trace_file`.

This is abnormal because the producer left no status and no history, and no trace was recorded, so the cause of the failure cannot be recovered from the halt note or the run's files.
It looks like a lyx problem (the Merge-In row failing before writing any status or trace), not a task-code failure.

What to do: find out why Merge-In produced no status; make the driver record a trace file and a more specific reason when a producer exits without a status, so the halt note is diagnosable.

Provenance: task `plan-gate-fixes`; notes: `loom-halt.md`.
