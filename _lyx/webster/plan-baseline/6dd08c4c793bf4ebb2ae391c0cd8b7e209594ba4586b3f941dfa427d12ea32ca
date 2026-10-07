# Card 1 — Non-waiting live-run probe on the shuttle attach seam

**Create:**
- `plan:internal/shuttleengine#Runner.ProbeGated` -> `func (r *Runner) ProbeGated(spec Spec, gate GateSpec) (*Run, bool, error)`

**Edit:**
- `internal/shuttleengine/attach.go`
- `internal/shuttleengine/attach_test.go`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#Runner.AttachGated`
- `internal/shuttleengine#Runner.reconstructAndWait`
- `internal/shuttleengine#Run.Wait`

**Intent:**
Add `Runner.ProbeGated`, the one shuttle addition the two-agent burler round needs (scope item "a non-waiting live-run probe").
It runs exactly `AttachGated`'s probe — the same spec normalization, candidate scan, reed gates with their `soleFinishedCandidate` harvests, dispositions, multiplicity refusals, and the superseded-strand removal on the not-found answer — but on a found candidate it returns the reconstructed `*Run` handle (fresh deadline, `gate` set on the handle) without calling `Wait`.
The caller waits on the handle later, so it can hold two probed runs and wait on both concurrently (D6's resume).

Split `reconstructAndWait` into a `reconstruct` step that builds the `*Run` (unchanged fields, unchanged logging) and the existing wait;
`attach` gains a flag (or returns the handle) so `Attach`, `AttachGated` and `AttachIfLive` keep their exact behaviour as "probe, then `Wait`", and `ProbeGated` is the same body stopping before `Wait`.
No other shuttle behaviour changes.

What the new verb lets through, and its bound: it reconstructs a handle over a live agent and hands it to the caller without observing it, so a caller that never waits leaves the run's record reading running;
that is bounded because it removes nothing beyond what `AttachGated` already removes, starts nothing, and a later `Wait` or attach over the same record classifies it exactly as today.

Test in `attach_test.go`, reusing `seedAttachRun`/`seedPresentReedState`: a live attachable candidate returns a non-nil handle with `found` true whose `StrandGUID()` and `RunDir()` name the candidate, with no wait having run, and whose `Wait` then classifies the run (done once its output file exists);
the not-found answer returns `(nil, false, nil)`.
The superseded-strand removal and the errored or doubly-attachable refusals run in the shared `attach` body this card does not change, and `TestAttach_RemovesSupersededStrands` and `TestAttach_Multiplicity` already cover them, so no probe-specific rows repeat them.
Update the attach section of `internal/shuttleengine/doc.go` to name the probe form.
The probe test is one new top-level function, so `cmd/lyx/testdata/test-budget.yaml` raises the `internal/shuttleengine` row by one (`TestTestBudget_PackagesStayWithinBudget` fails on a package over its row).

**ImpactSummary:** additive method on `Runner`; `Attach`/`AttachGated`/`AttachIfLive` keep their behaviour through a shared reconstruct step, and the attach test, the shuttle doc and the budget row follow it.
