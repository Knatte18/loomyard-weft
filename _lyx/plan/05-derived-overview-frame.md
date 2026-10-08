# Card 5 — a state without an overview frame hash derives it from the plan baseline

**Create:**
- `plan:internal/websterengine#recordedOverviewFrame` -> `func recordedOverviewFrame(st *State, websterDir string) (string, error)`

**Edit:**
- `internal/websterengine#overviewIndexOnly` -> `func overviewIndexOnly(recorded, planDir string) (bool, error)`
- `internal/websterengine#overviewRefusal` -> `func overviewRefusal(recorded string) error`
- `internal/websterengine#fingerprintMismatchWayForward` -> `func fingerprintMismatchWayForward(st *State, planDir, websterDir, reentry string) string`
- `internal/websterengine#PlanEditError` -> `func PlanEditError(st *State, planDir, websterDir string) error`
- `internal/websterengine#batchCardEditError` -> `func batchCardEditError(st *State, bs *BatchState, b batcher.Batch, planDir, websterDir string) error`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#BeginBatch`
- `internal/websterengine#RecordBatch`
- `internal/websterengine#PersistRecoveryTerminal`
- `internal/websterengine#Run`
- `internal/webstercli#websterCLI.validateCmd`
- `internal/websterengine/fingerprint_test.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/webstercli/cli_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D3, the rebaseline half of #463: a run recorded before `PlanOverviewFrameHash` existed refuses every overview change, a Card-Index-only one included, so a follow-up card could not land without a reset.
The recorded frame now falls back to the frame of the stored plan-baseline copy of the overview.

`recordedOverviewFrame`, in `rebaseline.go`, returns the run's recorded overview frame hash:
- `st.PlanOverviewFrameHash` when set;
- otherwise, when `st.PlanFileHashes` names a hash for `00-overview.md` and the store under websterDir holds that copy (`planBaselinePath`), the frame hash of the copy's bytes, computed by `overviewFrameHashOf`;
- otherwise `""`, the run having no recorded frame: no hash recorded, the copy absent, or a copy without a parseable Card Index (an `overviewFrameHashOf` error).
A read failure other than an absent copy is returned as an error.
It derives no path: websterDir is the told `Geometry.WebsterDir` (`PATTERN-told-geometry`).

The re-signs thread the recorded frame or websterDir to where it is judged:
- `overviewIndexOnly` takes the recorded frame instead of the state: `""` reports false, otherwise the current file's frame hash is compared with it.
- `overviewRefusal` takes the recorded frame: `""` keeps today's no-frame refusal with its reset-to-start way forward, and a recorded or derived frame gives the outside-the-Card-Index refusal with the follow-up-card way forward.
- `Rebaseline` computes `recordedOverviewFrame(deps.State, deps.Geom.WebsterDir)` when `00-overview.md` changed, an error taking the same transient way forward as an `overviewIndexOnly` error, and passes the frame to both.
  A successful rebaseline records `PlanOverviewFrameHash` through `restampBaseline` as today, so the derivation runs only until the first restamp.
- `fingerprintMismatchWayForward` gains websterDir and uses the recorded frame for its overview branch, so the plan-drift advice offers `lyx webster rebaseline` for a Card-Index-only change in a state with no frame hash; a `recordedOverviewFrame` error falls back to the generic text as an `overviewIndexOnly` error does.
- `PlanEditError` and `batchCardEditError` gain websterDir and pass it on.
  Their callers pass the told webster dir: `BeginBatch`, `RecordBatch` and `PersistRecoveryTerminal` their geometry's `WebsterDir`, `Run` its `deps.Geom.WebsterDir`, and `websterCLI.validateCmd` its `c.geom.WebsterDir`.

The bound: the fallback accepts only an overview whose frame equals the frame of the stored copy the run recorded, and never accepts a frame it cannot compute; a record with no `PlanFileHashes` skips the changed-file check as today.

Tests:
- `rebaseline_test.go`, in `TestRebaseline_EditedPlan`: a state with `PlanFileHashes` and the overview's baseline copy but no `PlanOverviewFrameHash` accepts a Card-Index-only overview change with follow-up cards named by `--card`, and records `PlanOverviewFrameHash`;
  the same state refuses an overview change outside the Card Index with the follow-up-card way forward, not the reset-to-start one;
  with the baseline copy absent, rebaseline refuses as today.
- `fingerprint_test.go`: the calls take the webster dir, and a row pins that `PlanEditError`'s advice for such a state, after a Card-Index-only change, offers `lyx webster rebaseline`.
- `internal/webstercli/cli_test.go`: `TestValidateCmd_RefusesOverviewEditWithoutRestamp`'s `PlanEditError` call passes `c.geom.WebsterDir`.

Docs: `doc.go`'s overview paragraph states that a state without `PlanOverviewFrameHash` takes the recorded frame from the stored baseline copy of the overview until the first restamp.
`refusal-spec.md`'s `rebaseline overview changed` row is reworded: `00-overview.md` changed outside its Card Index, judged against the recorded frame or the frame derived from the stored baseline copy, or changed at all in a run where neither yields a frame.

**ImpactSummary:** five unexported or exported plan-edit helpers take the webster dir; every caller in `websterengine` and `webstercli`'s validate verb passes its told `WebsterDir`; behavior changes only for a state without a frame hash.

**Commit:** 5: webster rebaseline: derive the recorded overview frame from the plan baseline copy (#463)
