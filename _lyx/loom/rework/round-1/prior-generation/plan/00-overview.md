---
format: 5
approved: true
language: go
---

# Plan: agent-self-reflect

Every loom run's friction reaches GitHub as issues with no operator in the loop, under `lyx shed step` / `lyx loom step` as well as `lyx loom run`.
Agents and Go write friction into the run's friction directory, one re-entrant reflection agent files it through `lyx selfreport create`, and nothing archives a note before a reflection has covered it.

## Card Index

1 — reflect-reentrant — `frictionengine.Reflect` records its covered notes, attaches to a live agent and archives only covered files
2 — step-after-hook — `shedverbs` gains an `AfterStep` hook, a `friction` key on every step envelope, and `run` error envelopes carrying `PostRun` extras
3 — reflect-under-step — the Friction-Reflect row reflects under `step`; a fakeable reflection shuttle seam
4 — refusal-note-writer — `websterengine.WriteRefusalNote` writes a webster refusal's friction note
5 — webster-refusal-notes — every `lyx webster` verb but `validate` notes its refusals
6 — halt-notes — a `blocked` or `failed` halt writes a Go halt note and reflects, under `run` and `step`
7 — drive-skill-filing — ly-drive writes friction into the friction directory and files nothing
8 — reflection-stencil — the reflection files every distinct lyx problem with provenance, and reports only after filing
9 — friction-capture-docs — the Friction Capture invariant and the overview and package docs

## Shared Decisions

### Decision: the board's post-Discussion additions are not carried

Board entry `agent-self-reflect` records two items added after the decision record was written: Tier 1 anomaly filing (`detectAndFileAnomalies`) under `step`, and a check whether `internal/loomengine/anomaly.go` loses a halt that is later resumed.
The decision record, this plan's answer key, keeps Tier 1 filing `run`-only under Scope Out and licenses no change to Tier 1's filing or retry, so no card carries either item.
Both need the decision record amended through Discussion before a plan can carry them.
Under `run`, `loomPostRun` keeps calling `detectAndFileAnomalies` as today, unchanged by this plan.

### Decision: a failed halt's error envelope carries the friction status

A `failed` halt always reaches the verb as a non-nil error from `shed.Step` or `shed.Run`, so it always prints an error envelope, and D2's acceptance criterion requires that envelope to carry the `friction` status as a `blocked` halt's success envelope does.
`AfterStep` runs on every `shed.Step` return, including errors, before the envelope is written, mirroring `PostRun`, and its return is the `friction` key on the success envelope and on every error envelope a `shed.Step` error produces.
The step error envelope's closed key set gains `friction` on every error envelope: a `PreStep` or `BuildShed` refusal, where no step ran, and a nil hook report it empty.
The generic `run` body merges `PostRun`'s extras onto the error envelope as onto the success envelope, so loom's `failed` halt under `run` reports its status too; `battenPostRun` returns nil on a non-nil `runErr`, so batten's error envelope stays as it is today.
Card 2 makes the generic change and card 6 returns the halt's status on the `failed` arms.

### Decision: one fakeable reflection seam

`loomCLI.reflectionShuttle` (card 3) is the `frictionengine.Shuttle` every reflection goes through, wired to the shuttle runner in production.
Cards 3 and 6 fake the reflection agent, and so its `lyx selfreport create` filing, through it with `shedfake.Shuttle`; no test spawns an agent or files an issue.

## verify:

```
go build ./...
go test ./internal/frictionengine/... ./internal/friction/... ./internal/shedverbs/... ./internal/loomcli/... ./internal/battencli/... ./internal/loomshed/... ./internal/loomengine/... ./internal/websterengine/... ./internal/webstercli/... ./internal/testkit/... ./contracts/stencils/... ./cmd/lyx/...
go test -tags integration ./internal/frictionengine/... ./internal/friction/... ./internal/shedverbs/... ./internal/loomcli/... ./internal/battencli/... ./internal/loomshed/... ./internal/loomengine/... ./internal/websterengine/... ./internal/webstercli/... ./internal/testkit/... ./contracts/stencils/...
go vet -tags smoke ./internal/loomcli/... ./internal/battencli/... ./internal/webstercli/... ./internal/shedverbs/... ./internal/frictionengine/...
```
