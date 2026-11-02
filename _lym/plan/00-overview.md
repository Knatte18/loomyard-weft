---
format: 5
approved: true
language: go
---

# Plan: one shared cold gate fixer for four gates

When a writer's gate fails, a fresh, cheap agent clears the mechanical failures before the writer hears of them.
A new package, `internal/gatefixer`, owns the life of one cold fixer strand: spawn-or-attach, the bounded wait, the stop and the runproc registration, its prompt rendered from one shared cold frame plus one caller body.
Webster's `lym webster fix-verify` moves onto it; the Darn-Write, Plan-Write and Discussion-Write row gates wrap their gate entry so a failure at the writer's idle boundary goes to a fixer and the gate is re-run; and an agent's red `lym gate test --gate card` call hands off to a fixer inside its detached runner.
Writers that loop on their own validator stop doing so when their row's fixer is on.

## Card Index

1 — gatefixer-core — new `internal/gatefixer`: fixer roles, record and store, spawn-or-attach, wait, stop, runproc registrar, and the PATTERN entry
2 — gatefixer-prompt — shared cold frame stencil and `RenderPrompt`
3 — webster-fixer-on-core — `fix-verify` and the plan fixer's wait on `gatefixer`; cold prefix and webster wait helpers deleted; verify fixer registered with runproc
4 — webster-file-headers — render.go, strand.go and recoverbatch.go file headers name the shared frame and gatefixer's members in place of the deleted webster ones
5 — gate-fixer-config — gate.yaml `fixer` on the `card` key and `fixer_timeout_min`
6 — module-fixer-keys — `darn.gate_fixer`, `tapestry.plan_gate_fixer`, `loom.discussion_gate_fixer`
7 — row-gate-wrap — `gatefixer.WrapRowGate` and the row fixer's record set
8 — shedrecipe-fixer-element — the `fixer` gate element key, the `Env.GateFixer` seam and the three rows' opt-in
9 — loom-row-fixer-wiring — loomcli wires the row fixer seam, the row visit directory and the three body stencils
10 — goto-ends-fixers — an admitted goto or targeted relaunch ends every fixer role under the pair before it writes the move
11 — writer-stencils-fixer-mode — Plan-Write and Discussion-Write prompts drop the validator loop when the fixer is on
12 — card-gate-front — the front carries the strand's role and keys a fixer's own call apart
13 — card-gate-runner-fixer — the detached runner's fixer phase, attach, re-run and join rule
14 — webster-fork-card-fixer-text — the Webster fork and recovery prompts name the card-gate fixer when it is on

## Shared Decisions

- `internal/gatefixer` imports nothing of `internal/websterengine`, so `internal/gatecli` reaches the fixer roles and the core without learning anything of webster (D4); webster's `StrandLive` and `TurnEndedAfter` move into it (cards 1 and 3) for that reason.
- A row fixer's wrap lives in `internal/gatefixer` and reaches the rows through one `shedrecipe` seam, `Env.GateFixer`, so `shedrecipe` imports nothing new and a nil seam leaves every gate as today.
- A row fixer's records live under `<run scratch dir>/gatefix/<row>-<history length>-<newest Repeats>/<entry>/`; an agent-call fixer's record lives in its detach key directory as `fixer.json`, with its report and reply in the run directory.
- Prompt prose lives in stencils read through `stencilstore` at call time: the shared frame and one body per caller under `contracts/stencils/gatefix/`.

## PATTERN changes

- `PATTERN-gate-fixer` — adds: Clearing a failed gate with a cold agent: only `gatefixer`, one fixer per failed evaluation; Go's re-run of the gate judges, never the fixer; a fixer's own gate calls spawn none.
