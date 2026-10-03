---
format: 5
approved: true
first_card: 10
language: go
---

# Plan: agent-self-reflect

Every loom run's friction reaches GitHub as issues with no operator in the loop, under `lyx shed step` / `lyx loom step` as well as `lyx loom run`.
Agents and Go write friction into the run's friction directory, one re-entrant reflection agent files it through `lyx selfreport create`, and nothing archives a note before a reflection has covered it.

The decision record's D1–D7 are built.
PR-Gate rejected PR 355 for the two operator additions after Discussion, which the decision record now carries: Tier 1 anomaly filing (`detectAndFileAnomalies`) also runs under `step`, and a failed Tier 1 filing is retried on every later pass, so a halt that is resumed is still filed.
This generation builds those two.

## Card Index

10 — anomaly-pending-retry — a failed Tier 1 filing stays pending in the filed-title marker and is retried on every later detection pass
11 — anomalies-under-step — `loomAfterStep` runs Tier 1 filing through a fakeable `loomCLI.fileIssue` seam, with the entry observation taken after the busy probe
12 — halt-filing-ownership — the reflection stencil, `CONSTRAINTS.md` and `docs/overview.md` state that Tier 1 files the halt event and the reflection files the lyx problems behind it

## Shared Decisions

### Decision: Tier 1 files the halt event, the reflection the lyx problems behind it

With the `selfreport` knob on, a halt is filed twice unless the two filers split it.
`detectAndFileAnomalies` runs first in both hooks, ahead of the halt note and reflection, and files the halt event under its stable anomaly title, deduplicated by the filed-title marker.
The halt note then says Tier 1 files the halt event, and the reflection stencil tells the agent to file only the lyx problems behind a halt whose note says so.
With the knob off the note carries no such line, and the reflection files the halt as it does today.

### Decision: one fakeable filing seam

`loomCLI.fileIssue` (card 11) is the filer both hooks pass to `detectAndFileAnomalies`, wired to `selfreportengine.CreateIssue` in production and to a refusing filer under `testing.Testing()`.
Card 11's tests fake it; card 10's tests keep driving `detectAndFileAnomalies` directly through `selfreportDeps.FileIssue`.
No test files a real issue.

### Decision: the pending list lives in the filed-title marker

A failed filing is kept as a title plus its rendered body in the existing machine-local marker, not in a new file, so the marker stays the one record of what Tier 1 has filed and still owes.
Losing the marker costs at most one duplicate issue and the pending retries with it, the trade the marker already accepts.

## verify:

```
go build ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
```
