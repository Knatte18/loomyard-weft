# Card 5 — Trace one line per plan: handle in every terminal pass

**Create:**
- `plan:internal/websterengine#bindCardNotDone` -> `const bindCardNotDone planindex.BindReason = "card-not-done"`
- `plan:internal/websterengine#bindBatchNotDone` -> `const bindBatchNotDone planindex.BindReason = "batch-not-done"`
- `plan:internal/websterengine#bindDeltaUnavailable` -> `const bindDeltaUnavailable planindex.BindReason = "delta-unavailable"`
- `plan:internal/websterengine#bindLanguageNone` -> `const bindLanguageNone planindex.BindReason = "language-none"`
- `plan:internal/websterengine#logBindOutcomes` -> `func logBindOutcomes(batch int, outcomes []planindex.BindOutcome)`
- `plan:internal/websterengine_test#fakeIndex.DoneChecks` -> `func (i *fakeIndex) DoneChecks(plan *planparser.Plan, cards []planparser.Card, worktreeRoot string) ([]planindex.Finding, error)`
- `plan:internal/websterengine_test#TestRecordBatch_BindTrace` -> `func TestRecordBatch_BindTrace(t *testing.T)`

**Edit:**
- `internal/websterengine#postBatchInputs`
- `internal/websterengine#postBatchChecks`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#PersistRecoveryTerminal`
- `internal/websterengine/doc.go`
- `internal/websterengine_test#fakeIndex`
- `internal/websterengine_test#fakeIndex.Delta`
- `internal/websterengine_test#fakeDelta`
- `internal/websterengine_test#fakeDelta.BindHandles`
- `internal/websterengine_test#TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/planparser#OwnedHandles`
- `plan:internal/planindex#BindOutcome`
- `internal/websterengine_test#captureLogs`

**Intent:**
Decision D2: every terminal pass logs, through `internal/logger` at Info, exactly one line per `plan:` handle the batch's cards own.
That makes the bind rate countable by grep.

Add `Number int`, the batch number, to `postBatchInputs`.
`RecordBatch` and `PersistRecoveryTerminal`, the struct's two call sites, set it.
Declare the four reasons `postBatchChecks` decides as `planindex.BindReason` constants beside it: `card-not-done`, `batch-not-done`, `delta-unavailable` and `language-none`.
Card 3 declared the other two.
`logBindOutcomes` writes one `logger.Info("webster: handle bind", ...)` line per outcome with these keys, so a grep counts them:
- `batch`
- `card`
- `handle`
- `outcome`: `bound` or `failed`
- `glyph` on a bound line, or `reason` on a failed line

`postBatchChecks` builds the owned handles of each card in `in.Cards` with `planparser.OwnedHandles`, and logs at these points:
- When the done-checks report findings, it logs the lines and then returns the not-done refusal as today.
  A card that a finding's `Card` names, matched by card ID, gets `card-not-done` for every handle it owns.
  Every other card of the batch gets `batch-not-done`.
- Under `language: none` (`Plan.GlyphLanguage` reports false), once `BindHandles` returns without error, every handle gets `language-none`.
  This replaces any other reason.
- When `Delta` failed with `planindex.ErrQuarryUnavailable`, once `BindHandles` returns without error, every handle gets `delta-unavailable`.
  The outcomes `BindHandles` computed over the empty delta are dropped, so no handle gets a second line.
- Otherwise it logs `BindHandles`' outcomes as returned, before the bind-count-mismatch refusal.

Some passes log no bind line, because the verb fails with that error first:
- the `in.Geom.index()` lookup errors
- `DoneChecks` errors
- `Delta` errors with anything other than `ErrQuarryUnavailable`
- `BindHandles` errors

The trace adds no finding, no refusal and no state.

Tests:
- Give `fakeIndex` a `doneErr` field.
  When it is set, its new `DoneChecks` override returns it.
  Otherwise the override forwards to the embedded index.
- Give `fakeIndex` a `bindErr` field, which `fakeIndex.Delta` copies into the `fakeDelta` it returns.
  `fakeDelta` gains the matching `bindErr` field, and its `BindHandles` returns that error, with no findings and no outcomes, when it is set.
  Otherwise it forwards to `planglyph.BindHandles` as today.
- `TestRecordBatch_BindTrace` is a new table test in `internal/websterengine/recordbatch_test.go`.
  It drives `RecordBatch` over the record fixture with `captureLogs`, at a verbosity that keeps Info lines, so it does not call `t.Parallel`.
  The fixture's own plan directory is empty and its batch holds one card, so a bound handle's `RewriteRefs` would fail on `ParsePlan` and no batch would mix two cards.
  Each row therefore writes its own plan with `plankit.Write` into a `t.TempDir()`, as `writeRecordPlanDir` does, and points `Deps.Geom.PlanDir`, `Deps.Plan`, `Deps.Batches` and batch 1's begun `Cards` at it.
  The mixed row's batch 1 holds two cards.
  The row then re-stamps the state with `websterengine.RestampPlanBaseline`, so `PlanEditError` accepts the plan, and sets `fakeIndex.delta` to the created symbols its handles match.
  For each row it asserts the exact set of `webster: handle bind` lines by card, handle, outcome and glyph or reason.
  The rows are:
  - all handles bound
  - a card whose only handle is unmatched logs `no-delta-match`
  - one handle unmatched, so its matched sibling logs `sibling-unmatched`
  - a done card and a not-done card in one batch log `card-not-done` and `batch-not-done`
  - an `ErrQuarryUnavailable` delta (`fakeIndex.deltaErr`) logs `delta-unavailable` once per handle and nothing else
  - a `language: none` plan logs `language-none` per handle
  - a card with no handles logs nothing
  - an `Index` lookup error (`Geom.Index` nil), a `DoneChecks` error (`doneErr`), a plain `Delta` error and a `BindHandles` error (`bindErr`) log no line
- Extend `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` so its unlanded card also declares a `plan:` handle.
  Assert that recover-batch logs that handle's `card-not-done` line, which shows the recovery path logs the same lines.

`TestRecordBatch_BindTrace` adds a top-level test, so raise the `internal/websterengine` row in `cmd/lyx/testdata/test-budget.yaml` by one.
In `internal/websterengine/doc.go`, add to the "every terminal batch runs the same mechanical pass" section:
- the trace line
- its keys
- the six reasons and when each applies
- the passes that log nothing

**ImpactSummary:** confined to `internal/websterengine`'s terminal pass shared by record-batch and recover-batch: new Info log lines and one new input field set by its two callers, with no change to any refusal, finding, warning or state write.
