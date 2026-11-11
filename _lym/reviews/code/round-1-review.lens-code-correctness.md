```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed branch with a concrete input.

**Per-branch list**

- `loadSkillTurn` cap check (`loadturn.go:154`): correct. At the 8th API-error turn end, `errors` is 8 and `pending` is true, so it fires after the model-rejected check and before `noticeLoadHold`. A read that finds a real turn end returns before the cap. A read carrying several errors still trips the cap through `>=`. The 7th error followed by a typed load gives `pending=false`, so no cap and the load is typed. `errors` is never reset inside one call. The cap therefore counts every API error in the call, not only consecutive ones. That fits "one hold" as documented.
- `loadAPIErrorCapError.Error()`: correct. Empty `text` falls back to `apiErrorUnknownText`, and the message names the count and the cap.
- `abandonLoad`:
  - Correct. A capped error maps to `OutcomeAPIError`, `classifyDeadlineExpiry` returns Done when all outputs exist, and otherwise `abandonStartup` gives `ErrNotStarted` wrapping the cap error.
  - `Result.APIError` is set afterwards, as on the rejected path.
  - The pane, strand and deadline extension match the rejected path. `endLoadHold` runs via `defer` and extends the deadline.
- `noticeLoadHold`: correct. The notice text names the cap through `%d` of the constant.
- `stepLocked` `startedAt` (`run.go:345`): correct. It is taken after the resume write and before `Call`, and every `appendHistory` path carries it. That covers done, stuck, awaiting, blocked, budget-capped, the failed callErr path and the unrecognised-outcome path. `rerouteStuck` receives it in all three of its entries. The goto and skipped entries (`goto.go:140,263`, `entry.go:55`) omit it by design.
- `idleGapsOf`:
  - Empty and single-entry histories return `[]`, so JSON gets `[]`, not null.
  - A gap of exactly 5m is reported and 4m59s is not.
  - A folded `prev` ends at its `LastAt`. A folded `next` uses its first call's `StartedAt`, so the gap into the fold is right.
  - A goto or skipped `next` uses its own `At`.
  - A `next` with no `StartedAt` fails the parse and ends no gap.
  - Out-of-order times give a negative idle, which is under the floor, so no gap.
- `RenderStatusHuman` idle line and the `idle_gaps` envelope key: correct. The test row and `doc.go` match the format.

**Findings**

1. LOW, design. `internal/shedverbs/status.go:124-142`, evidence `run.go:378-383` and `appendOrFold`.
   - Idle time between repeated calls folded into one entry is invisible. The fold keeps the first call's `At` and `StartedAt`. A budget-exempt Stuck that repeats hours apart yields one entry, so the `LastAt` to next `StartedAt` gap is all that shows.
   - The idle inside the fold is lost. That is the "stuck step with hours around it" case from the Goal.
   - Fix: either record the per-call gaps at fold time, or state in `doc.go` and `IdleGap` that gaps inside a folded entry are not reported.
2. NIT, design. `status.go:124`: the tail gap, from the last entry's end to now on a halted or waiting run, is never reported. The Goal says "between a stuck step and the next step", so there is no next step to measure to. This is not a defect; I mention it only because the focus file asks.

**Not a finding, but unverified**

- `readLoadTurn` calls `apiErrorRead` from both the `SessionSignalAPIErrorTurnEnd` signal and the `TurnEndActivity.APIError` event. If one engine emitted both for the same turn end, `errors` would double and the cap would fire at the 4th real error. My context cannot show whether an engine does this. It was already a risk for backoff and `retypeAt`, and the new test uses one source only.

**Tool calls (1)**
1. grep `classifyDeadlineExpiry` and `abandonStartup` in `wait.go`: my context lacked their bodies, to confirm the all-outputs-exist branch and the finalize order.
