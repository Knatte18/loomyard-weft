---
format: 5
approved: true
language: go
---

# Plan: shuttle-gate-early-done

Shuttle must not take a writer's turn as over while the model is still working on it, and a twill Plan-Write must not end Done before its lens review is recorded.
In both bench runs a task-notification turn that started right after a turn boundary was released by the 10 s idle-pane override while the model was thinking, and the Plan gate passed.
Cards 1 and 2 give shuttle an engine reading of whether an unmatched turn start is still in flight, and let that reading veto the override (D1, D2).
Cards 3 to 6 record a run's fork authorization at spawn, select a lens-review-requiring plan gate from it for the Plan-Write row only, and pair that gate with `lyx loom validate-plan --lens-review` (D3).
Card 7 makes the twill Plan writer run that mode as its final self-check (D4).

## Card Index

1 — shuttle-turn-in-flight-veto — an unmatched turn start the engine reads in flight vetoes the idle-pane override, unless a session start follows it
2 — claudeengine-turn-progress — Claude reads a turn start's progress from the newest main-chain transcript entry
3 — shuttle-recorded-fork-authorization — the run record carries fork authorization from spawn, and the fork audit and a gate entry's Forked closure read it
4 — loomshed-lens-review-gate — ValidatePlanLensReview and its gate add a presence check on lens-review.md to the plan gate's set
5 — shedrecipe-plan-write-forked-gate — only Plan-Write's plan gate entry carries the lens-review closure
6 — validate-plan-lens-review — the --lens-review self-check mode, its flag refusals and its parity with the gate
7 — twill-final-self-check — the twill directive's step 8 takes its own command, --lens-review for the Plan host

## Shared Decisions

- `TurnProgress`'s zero value is `TurnProgressUnknown`, so an engine or fake without `TurnProgressReader`, and every existing test row, keeps today's override.
- The lens-review closure rides `GateEntry.Forked`, chosen by `Run.evaluateGate` from `RunState.ForkSubagents`; no gate signature changes, and `Final` keeps precedence over it.
- Plan-Write is told apart by its dedicated constructor (`planWriteEntry`), never by `resolveGateSpec` switching on a row name.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./internal/shuttleengine/... ./internal/loomshed ./internal/loomcli ./internal/shedrecipe ./internal/loomengine ./internal/twilldirective ./contracts/stencils ./contracts/specs ./cmd/lyx
go vet -tags tmux ./internal/shuttleengine/... ./internal/loomshed ./internal/loomcli ./internal/shedrecipe ./internal/loomengine ./internal/twilldirective ./contracts/stencils ./contracts/specs ./cmd/lyx
go vet -tags llm ./internal/shuttleengine/... ./internal/loomshed ./internal/loomcli ./internal/shedrecipe ./internal/loomengine ./internal/twilldirective ./contracts/stencils ./contracts/specs ./cmd/lyx
go test ./internal/shuttleengine/... ./internal/loomshed ./internal/loomcli ./internal/shedrecipe ./internal/loomengine ./internal/twilldirective ./contracts/stencils ./contracts/specs ./cmd/lyx
go test -tags integration ./internal/shuttleengine/... ./internal/loomshed ./internal/loomcli ./internal/shedrecipe ./internal/loomengine ./internal/twilldirective ./contracts/stencils ./contracts/specs ./cmd/lyx
```
