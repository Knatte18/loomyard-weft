# Card 3 — rebaseline-refusals-keyed

**Edit:**
- `internal/websterengine#RebaselineDeps`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#rebaselineBatches`
- `internal/websterengine#recordedOverviewFrame` -> `func recordedOverviewFrame(st *State, websterDir string) (frame, cause, path string, err error)`
- `internal/websterengine#overviewRefusal` -> `func overviewRefusal(recorded, cause, path, step string) error`
- `internal/websterengine#fingerprintMismatchWayForward`
- `internal/websterengine#CheckBatchOrder` -> `func CheckBatchOrder(batches []batcher.Batch, step string) error`
- `internal/websterengine#formBatches`
- `internal/websterengine#ExecutionBatches`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/sequence_test.go`
- `internal/websterengine/doc.go`
- `//contracts/specs/refusal-spec.md`

**Create:**
- `plan:internal/websterengine#followUpLanding` -> `func followUpLanding(step string) string`

**Uses:**
- `internal/websterengine#batchCards`

**Intent:** Key every clause of a rebaseline refusal on the step the caller names, and name the cause of an empty overview frame (D1 and D3).
`RebaselineDeps` gains `Step`, the step its refusals name: empty means `lyx webster run`, as `RunDeps.ReentryStep` does, and the loom's Webster row sets its re-step text (card 4).
Every clause of a `Rebaseline` refusal that names a manual verb today is keyed the same way: the follow-up-card landing, the changed-but-not-named card set, each transient retry, and every fresh-restart route that ends in `freshRestartSteps(stepRun)`, namely the no-frame `overviewRefusal`, the begun batch's card-set change in `rebaselineBatches` and both branches of the changed and done-changed way forward, which pass the step to `freshRestartSteps` in place of `stepRun`.
`CheckBatchOrder`, which `rebaselineBatches` calls on the edited plan's batches, takes the step too: empty keeps today's text naming `lyx webster rebaseline --card NN` and `lyx webster run --fresh`, and a set step says to fix the plan's card order and re-step the row; `formBatches` and `ExecutionBatches` pass the empty step, and `rebaselineBatches` passes the deps' step on both of its branches: its no-partition branch, which today calls `formBatches` and so would check the order with the empty step, calls `batchCards` with the identity batchifier and then `CheckBatchOrder` with the deps' step in its place, so an order violation on a record without a partition names the re-step under the loom too.
With an empty `Step` every text is today's, the `restampBaseline` failure included; with a step set, the landing says to fix the plan, move a done card's edit to a follow-up card after the last begun batch with its Card Index line in `00-overview.md`, and re-step the row, the unnamed-card clause says to re-step the row after fixing the plan, a transient says to re-step the row, and a fresh restart says `lyx webster reset --to start` then re-step the row, so no step-set text names `lyx webster run` or `lyx webster rebaseline`.
A `restampBaseline` failure, today returned bare from `Rebaseline` with its own `transient, re-run the verb` clause, stays bare with an empty `Step`, and with a step set `Rebaseline` wraps it with the keyed re-step clause like the other transients.
`followUpLanding` renders the landing for a step; the `followUpCardLanding` constant stays as the manual text `fingerprintMismatchWayForward` and `followUpLanding("")` return.
`recordedOverviewFrame` returns the cause of an empty frame beside it, one of three fixed texts, no overview hash recorded, the baseline copy absent, or a copy without a parseable Card Index, and the baseline copy's path where there is one; the helper logs nothing.
`Rebaseline` logs the cause at Info at its own call, with the path when there is one, so a rebaseline logs it once; `fingerprintMismatchWayForward`, its other caller, drops the new returns and keeps its no-frame text unchanged.
`overviewRefusal`'s no-frame text keeps its `recorded no overview frame` phrase, which `TestRebaseline_EditedPlan`'s recorded-frame row asserts absent, and adds the cause and the path, and the refusal-spec row `rebaseline overview changed` carries the new wording; the other rebaseline rows and the batch-order row carry the step-keyed alternative beside today's text.
`doc.go`'s rebaseline passage states the logged empty-frame cause and that the refusals name the caller's step.
`TestRebaseline_OverviewFrameUnreadable` holds the rows the record's acceptance names, once each for no recorded overview hash, an absent baseline copy and a copy without a Card Index, each refusing with the cause, the last two naming the path, and each asserting exactly one captured Info line.
The absent-copy row is not new: `TestRebaseline_EditedPlan`'s row `a state without the overview frame hash and without the baseline copy refuses any overview change` already reaches that cause, so it moves into `TestRebaseline_OverviewFrameUnreadable`, which can capture the log, and its wanted text gains the cause and the baseline copy's path beside `recorded no overview frame` and the fresh-restart steps; the other two causes are the new rows.
The log capture swaps the logger's process-global output, so `TestRebaseline_OverviewFrameUnreadable` drops `t.Parallel` for itself and its rows under a comment naming that output, as `TestRun_FreshOverPendingFindings` does, and its doc comment widens to an empty recorded frame beside an unreadable overview.
It also gains a step-set row for its overview-read transient, the Card Index lost after the parse, asserting the re-step clause.
`TestRebaseline_CardSet` gains a step-set row per keyed clause, the mixed done-and-card-set case, a follow-up card ordered before a card it uses, and the same order violation over a record that holds no partition among them, plus the changed-file read and card-hash read transients, each forced by an unreadable card file, and every step-set row of both tests asserts the text contains neither `lyx webster rebaseline` nor `lyx webster run`.
`TestCheckBatchOrder` gains a step-set row naming the re-step and neither manual verb.

**ImpactSummary:** Websterengine only: two helpers and CheckBatchOrder re-signed, with their callers (fingerprintMismatchWayForward, Rebaseline, rebaselineBatches, formBatches, ExecutionBatches) on this card; the manual refusal texts and the CLI's rebaseline verb are unchanged.
