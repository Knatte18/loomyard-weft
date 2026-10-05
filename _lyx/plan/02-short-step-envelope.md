# Card 2 — lyx shed step prints a short envelope; the record keeps the full one

**Edit:**
- `internal/shedverbs/step.go`
- `internal/shedverbs/steprecord.go`
- `internal/shedverbs/verbs.go`
- `internal/shedverbs/doc.go`
- `internal/shedverbs/steprecord_test.go`
- `internal/shedcli/parity_test.go`
- `internal/loomcli/cli.go`
- `pattern/PATTERN-shed-verb-set.md`

**Uses:**
- `internal/output/output.go`
- `internal/shedverbs/step_test.go`
- `internal/loomcli/step_test.go`

**Intent:**
Decision record D2: the generic `step` verb prints one short JSON line by default, writes the full envelope to its per-invocation record first, and gains `--full`.

Full envelope: `StepEnvelope` and `stepErrFields` keep building today's closed key sets unchanged; `TestStepEnvelope_KeySetIsClosed` and `internal/loomcli/step_test.go` keep passing untouched.

Record-first write (`steprecord.go`): replace `stepRecorder.tee` and `stepRecorder.finish` with one method that writes given envelope bytes to `<StepsDir>/<trace_id>.json` and reports the path written, or failure.
A disabled recorder (empty `Spec.StepsDir` or trace id) and a failed write both report failure, the write failure still logged by `logger.Warn` as today.
`begin` and the in-flight record are unchanged, and the record path stays the one `lastStepOf` reports as `last_step.envelope_path`.
Update the file header: the record holds the full envelope, written before stdout is printed.

`stepCmd` (`step.go`): every exit path, success and each refusal, renders the full envelope through `internal/output` into a buffer (`output.Ok` or `output.ErrFields`, keeping their exit code), hands those bytes to the recorder, and only then prints to stdout:
- the record was written and `--full` is not set: the short envelope, through the same `output` function so `ok`/`error` keep their meaning;
- `--full` is set, or no record was written: the full envelope bytes, byte-identical to the record.
The exit code is the full envelope's in every case.
Short keys, built in one helper per envelope kind from the full map plus the record path, with a comment naming each closed set:
- success: `run_id`, `producer`, `outcome`, `state`, `continue`, `reason`, `output`, `next`, `progress`, `history_length`, `trace_file`, `envelope_path`, plus `friction` and `parent_notice` only when non-empty;
- error: `kind`, `transient`, `run_id`, `trace_file`, `envelope_path`, plus `friction` only when non-empty, beside `output.ErrFields`'s own `error`.
`envelope_path` is the path the recorder returned, built only from the told `Spec.StepsDir` (`PATTERN-shed-verb-set`).
Register `--full` on the command `stepCmd` returns (bool, help: print the full envelope on stdout instead of the short one; the record, exit code and run state are unchanged).
Update `StepEnvelope`'s doc comment: it is the full envelope, held by the step record and printed under `--full` or when no record is kept.
`StepKinds` and the derivation of `continue` are unchanged.

Bound: `--full` changes only what stdout prints; the record, the exit code and the run's state are the same with or without it.
The short line never names a missing file: without a record written, stdout carries the full envelope.

`verbs.go`: the `Verbs` comment's list of flags the generic bodies register gains step's `--full`.
`doc.go`: add the short envelope, the record-first write and `--full` to the package doc.
`internal/loomcli/cli.go`: the `loom step` `Long` text says the printed envelope is short, names `trace_file` and `envelope_path` (the full envelope, holding `friction_dir`, `scratch_dir` and `next_interrupt_policy`), and mentions `--full`; it stops claiming every error envelope carries `friction_dir` and `scratch_dir`.
Grep the repository for the old `Long` sentences ("it also names", "Every error envelope carries the same three keys") before rewording, and add any test asserting them to this card.
`pattern/PATTERN-shed-verb-set.md`, `## Step envelope`: the closed key set is the full envelope's, held in the step record; stdout prints the short envelope by default, whose two closed key sets are stated in `internal/shedverbs`, enforced by `internal/shedverbs/steprecord_test.go` beside `step_test.go`.
No `PATTERN.md` line changes.

Tests (`steprecord_test.go`):
- Replace `TestStepCmd_EnvelopeRecordEqualsPrintedLine` with one table over success and refusal, each run without and with `--full`, each with its own trace id where the logger allows, or its own steps dir:
  stdout without `--full` decodes to exactly the short key set for its kind (`ok`/`error` included), with `envelope_path` equal to `<stepsDir>/<trace_id>.json`;
  the record decodes to the full key set (`StepEnvelope`'s keys plus `ok`, or `stepErrFields`'s keys plus `ok` and `error`);
  with `--full`, stdout bytes equal the record bytes, and the record's decoded content equals the run without `--full`;
  the exit code is the same in both runs.
- `TestStatusCmd_LastStep`, or the table above: after a recorded step, `statusCmd` over the same `StepsDir` reports `last_step.envelope_path` equal to the short envelope's `envelope_path`.
- `TestStepCmd_NoStepsDirKeepsNoRecords` and `TestStepCmd_RecordWriteFailureLeavesEnvelopeUnchanged`: also assert stdout is the full envelope (e.g. `status_file`, `scratch_dir` and `trace_id` present, no `envelope_path`).
- `internal/shedcli/parity_test.go` (`TestParity_LoomStep_RunLockBusy`, `TestParity_BattenStep_RunLockBusy`): the busy refusal in a real hub keeps a record, so its stdout no longer carries `scratch_dir`/`friction_dir`; read those two keys from the file at the envelope's `envelope_path` instead, and assert `envelope_path` is non-empty on both paths.
  The existing step tests with no `StepsDir` print the full envelope and keep passing unchanged.

**ImpactSummary:** `lyx shed step`, `lyx loom step` and `lyx batten step` print the short envelope by default and accept `--full`; the step record now holds the full envelope rather than a copy of stdout; no Go caller parses the dropped stdout keys outside the updated tests.

**Commit:** 2: shed step: short envelope on stdout, full envelope in the record, --full flag
