---
format: 5
approved: false
language: go
---

# Plan: agent-self-reflect

Every loom run's friction reaches GitHub as issues with no operator in the loop, under `lyx shed step` / `lyx loom step` as well as `lyx loom run`.
Agents and Go write friction into the run's friction directory, one re-entrant reflection agent files it through `lyx selfreport create`, and nothing archives a note before a reflection has covered it.
Tier 1 anomaly filing also runs under `step`, by an operator addition recorded below.

## Card Index

1 — reflect-reentrant — `frictionengine.Reflect` records its covered notes, attaches to a live agent and archives only covered files
2 — step-after-hook — `shedverbs` gains an `AfterStep` hook and a `friction` key on the step success envelope
3 — reflect-under-step — the Friction-Reflect row reflects under `step`; a fakeable reflection shuttle seam
4 — refusal-note-writer — `websterengine.WriteRefusalNote` writes a webster refusal's friction note
5 — webster-refusal-notes — every `lyx webster` verb but `validate` notes its refusals
6 — halt-notes — a `blocked` or `failed` halt writes a Go halt note and reflects, under `run` and `step`
7 — anomalies-under-step — Tier 1 anomaly detection and filing run under `step`
8 — anomaly-pending-retry — an anomaly whose filing failed is retried after its halt is resumed
9 — drive-skill-filing — ly-drive writes friction into the friction directory and files nothing
10 — reflection-stencil — the reflection files every distinct lyx problem with provenance, and reports only after filing
11 — friction-capture-docs — the Friction Capture invariant and the overview and package docs

## Shared Decisions

### Decision: operator additions relayed by the hub after Discussion

Two scope items were added by the operator after the decision record was written, relayed by the hub orchestrator (`ly:orch`) and carried on board entry `agent-self-reflect`.
They are deliberate scope, not drift from the decision record:

1. Tier 1 anomaly filing (`detectAndFileAnomalies`) also runs under `step`, overriding the record's Out item "Tier 1 anomaly filing stays `run`-only": the operator's no-approval-gate decision covers every filing path.
   Its existing skips (knob off, `ErrShedBusy`, cancelled context) stay.
   Card 7.
2. From the board's "Friction on halt" section (moved from `convergence-plan`): check whether `internal/loomengine/anomaly.go` loses a halt that is later resumed, and fix it if so.
   Card 8; the finding is below.

### Decision: resumed-halt loss in Tier 1 — the finding

`loomengine.DetectAnomalies` derives the halt-kind anomaly from the final status alone (`detectHaltAnomaly` reads `final.State` and `final.Error`), and `detectAndFileAnomalies` re-reads that status once, when the invocation ends.

- Under `run`, a halt ends the `shed.Run` call, so the status read at `loomPostRun` is the halt and the anomaly is detected then; resuming later does not lose a detected halt.
- Under `step`, nothing called detection before this plan, so every stepped halt was lost, resumed or not; card 7 runs detection after every step, which detects a halt at the step that reaches it.
- In both, `runFilingPass` leaves a title whose filing call failed unrecorded "to be retried next run", but the next pass re-detects only from the then-final status: once the halt is resumed and the run moves on, it is never detected again, and the issue is lost.
  This is the loss card 8 fixes, by keeping failed anomalies as pending entries in the filed-title marker and retrying them on every later pass.

### Decision: who files a halt

A halt can reach GitHub through two filers, and each owns one part.
Tier 1 owns the halt event: one `loom anomaly:` issue per halt title, deduplicated by its filed-title marker.
The reflection owns the friction notes, the Go halt note included.
When the `selfreport` knob is on, the halt note says Tier 1 files the halt itself, and the reflection stencil tells the agent to file only the distinct lyx problems behind it; with the knob off, the reflection owns the halt note as well.

### Decision: the step error envelope stays extras-free

`AfterStep` runs on every `shed.Step` return, including errors, before the envelope is written, mirroring `PostRun`.
Its returned friction status reaches the success envelope only: a step error envelope keeps its closed key set, as `run`'s error envelope carries no `PostRun` extras.
A `failed` halt therefore still writes its halt note and reflects before the error envelope is printed, and the envelope reports the failure, not the reflection status.

### Decision: one fakeable reflection seam

`loomCLI.reflectionShuttle` (card 3) is the `frictionengine.Shuttle` every reflection goes through, wired to the shuttle runner in production.
Cards 3, 6 and 7 fake the reflection agent through it with `shedfake.Shuttle`, and fake Tier 1 filing through `loomCLI.fileIssue` (card 7); no test spawns an agent or files an issue.

## verify:

```
go build ./...
go test ./internal/frictionengine/... ./internal/friction/... ./internal/shedverbs/... ./internal/loomcli/... ./internal/loomshed/... ./internal/loomengine/... ./internal/websterengine/... ./internal/webstercli/... ./internal/testkit/... ./contracts/stencils/...
go test -tags integration ./internal/frictionengine/... ./internal/friction/... ./internal/shedverbs/... ./internal/loomcli/... ./internal/loomshed/... ./internal/loomengine/... ./internal/websterengine/... ./internal/webstercli/... ./internal/testkit/... ./contracts/stencils/...
go vet -tags smoke ./internal/loomcli/... ./internal/webstercli/... ./internal/shedverbs/... ./internal/frictionengine/...
```
