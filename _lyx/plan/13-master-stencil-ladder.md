# Card 13 — master-stencil-ladder

**Edit:**
- `contracts/stencils/webster/webster-template-master.md`
- `internal/websterengine/template_test.go`

**Uses:**
- `internal/webstercli/recordbatch.go`

**Intent:**
Teach Master the new ways forward, so the Go-side dispositions of cards 8–12 are taken in-session rather than ending every run stuck.

In `webster-template-master.md`:

- Progress trail: add `failed` → webster rejected that batch's report; call `lyx webster recover-batch <NN>` and follow the failure ladder, exactly as for `stuck`.
- Failure ladder: add `record-batch` refusing with `{"batch_failed": true}` → call `lyx webster recover-batch <NN>` (the batch is already terminal-failed and its report archived), then follow the recover-batch rungs;
  and `{"report_archived": true}` → call `lyx webster begin-batch <NN>` and re-fork that batch from its fresh prompt.
- Replace "A card-not-done refusal ends your run as stuck" with the `batch_failed` rung above: a done-check failure now arrives as `batch_failed`.
- Replace "A policy violation ends your run as stuck" with: record-batch and run audit every transcript;
  a policy finding comes back on the envelope's `warnings` and the batch still records, so keep going;
  a correctness finding comes back as `batch_failed` and goes to recover-batch;
  still never work around an audit, and never write outside your two contract files.
  Remove "once a violation exists it will fail every later call too".
- The plan-drift section keeps `outcome: stuck`, and adds that the operator's way forward is to edit the plan and run `lyx webster rebaseline` before re-running.

`template_test.go`: update the assertions pinning the removed sections, and add assertions that the rendered master prompt contains `"batch_failed": true`, `"report_archived": true`, the `failed` progress rung, and `lyx webster rebaseline`.

**ImpactSummary:** Rewrites three sections of Master's stencil; only Master sessions read it, and deployed copies refresh through the stencil store.
