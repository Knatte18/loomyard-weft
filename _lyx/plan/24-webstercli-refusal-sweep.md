# Card 24 — webstercli-refusal-sweep

**Edit:**
- `internal/webstercli/beginbatch.go`
- `internal/webstercli/recordbatch.go`
- `internal/webstercli/recoverbatch.go`
- `internal/webstercli/run.go`
- `internal/webstercli/validate.go`
- `internal/webstercli/verbs_test.go`

**Intent:**
Give every webster CLI-level refusal a way forward (D1) and a reaching test, in the trailing `way forward: …` form card 23 uses.

Messages to fix:

- Every `… but the fabric sync failed` refusal (begin-batch, record-batch, recover-batch spawn and terminal, run): the state is saved locally; `lyx fabric sync` commits and pushes it, or the next bracket verb's own sync carries it.
- `run`, standalone reed boot failure, and `recover-batch`'s same failure: transient; re-run the verb.
- `recover-batch`, state.json vanished during the wait: re-run `lyx webster recover-batch NN`.
- `validate`, the re-baseline persistence failure: re-run `lyx webster validate`, or `lyx webster rebaseline`.
- The combined "persisting the plan-fingerprint re-baseline … failed" messages in begin-batch and record-batch: re-run the same verb; the re-baseline is recomputed from the plan on disk.

Already carrying a way forward: `no run in progress; run "lyx webster run" first`, and run's `--plan-dir` override refusal.
They get a reaching test where none exists.

Grouped: a non-numeric batch argument (usage), and wiring failures in the pre-run.

Tests in `verbs_test.go` (integration): for the fabric-sync rows, a hub fixture whose fabric push is made to fail, then `lyx fabric sync` (or the next verb) succeeding;
for the vanished-state row, the verb re-run succeeding once state.json is back;
for `no run in progress`, `lyx webster run`'s state init followed by the refused verb succeeding.
Every hub fixture comes from `internal/hubforge`.

**ImpactSummary:** Message text and tests only in webster's CLI verbs; envelopes keep their keys.
