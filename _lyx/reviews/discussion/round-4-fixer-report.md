# Round 4 fixer-report — discussion

## Changes

- **F1** — fixed differently.
  D7's wiring builds the instruction-file names lazily and refuses nothing for the engine name.
  An unregistered Master engine is refused only by the `MerriamBase` callers, which form a fresh partition or rebaseline: `run`'s partition formation and auto-rebaseline, `webstercli`'s `executionBatches` with no state, and `rebaseline`.
  The way forward names `master` in `webster.yaml` and `lyx config webster`.
  Not made as proposed: the claim that `validate` and an await are never refused by the Master key.
  `internal/webstercli/validate.go:104` and `:280` call `executionBatches(plan, nil)` when no state exists and for `--batches`, and `internal/webstercli/awaitbatch.go:63` passes a nil state from `LoadState` too, so each reaches `MerriamBase`.
  The never-refused list and the new acceptance criterion are therefore scoped to a run with a state and to `validate` without `--batches`.
  `run`'s auto-rebaseline (`internal/websterengine/autorebaseline.go:49`) is added to the callers, since it also calls `MerriamBase`.
  D6's sentence that the lookup refusals are the only ones a spawn-free verb can hit now carves out `rebaseline` and `validate --batches`, and points to D7.
- **F2** — fixed as proposed.
  `Run`'s check is an entry gate beside the plan-approved and `deps.Batcher` checks, before `AcquireStateMutation`.
  recover-batch's check is the first step of the spawn branch, after the return to a live recovery (`recoverbatch.go:455`) and ahead of `AcceptBatchFabricReference`, `archiveStaleReport` and the prior strand's `StopStrand`.
  An acceptance criterion pins that a refused `run` (also `--fresh`) and a refused `recover-batch` leave state, the archive, the prior report and live strands unchanged.
- **F3** — fixed as proposed: the checks sit in the engine functions that spawn, `run`'s recovery check is a pre-flight, and the Bound says the wiring checks no engine.
- **F4** — fixed as proposed: the turn-end read resolves through the last `RecoverySessions` entry's `recoveryEngines` key, and a missing key is an engine-less entry.
- **F5** — fixed as proposed: the Bound gives the audit-error outcome (`claudeengine`'s parent-transcript read error, `audit.go:290`) with its way forward, and the empty-audit outcome; the open-risk entry is aligned with it.
- `support-log.md` gains a Round 4 section.

## Disputed

None.

## Deferred

None.
