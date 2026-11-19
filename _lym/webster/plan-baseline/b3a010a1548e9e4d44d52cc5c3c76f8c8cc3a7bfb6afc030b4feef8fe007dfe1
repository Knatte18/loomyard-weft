---
format: 5
approved: true
language: go
---

# Plan: Hardener, an autonomous behaviour-review campaign that replaces crucible

Build the `hardener` loom recipe: a board task's run that drives a live-substrate lym module against throwaway sandbox hubs round after round, through the Bouncer, the burler review row and a new `HardenFix` row carrying a Go-run live gate, and lands the fixes through Finalize.
The cards add the gate machinery first (`--copies`, the gate keys), then the `hardener.yaml` unit and the `internal/hardenshed` engine pieces, then the rows, the recipe and loom's wiring, the CLI verbs and the teardown hook, and finally retire treadle and crucible.

## Card Index

1 — gate-copies — add `lym gate test --copies` and the started-binaries line
2 — gate-harden-keys — ship the harden-live, harden-live-copies and harden-burler-fix gate keys and the hub-key lookup
3 — hardener-config — add the hardener.yaml config unit in the new hardenshed package
4 — loom-contract-segmented-build — let the loom contract accept a build row inside a review segment
5 — campaign-preflight — Preflight's hardener refusals and the cyclic model rotation
6 — live-gate — the live gate and its round live report
7 — sandbox-builder — build and list throwaway sandbox hubs under the run's sandbox root
8 — sandbox-reaper — the sandbox reaper and the PATTERN-child-processes change
9 — round-row — reap around each round row, write the round's Go files and archive the round
10 — bouncer-harden-seams — the Bouncer's facts section, red-live auto-accept bound and halt seam
11 — harden-rows — the HardenFix engine, the harden_round keys and the harden commit seam
12 — harden-rubric — the harden-rubric-review stencil
13 — hardener-recipe — the hardener recipe and its place in every closed recipe list
14 — loom-harden-wiring — wire the Harden perch, the campaign seams and the hardener tree scope in loom
15 — finalize-reap — Finalize reaps the sandbox root before it lands
16 — harden-cli — the lym harden sandbox verbs
17 — teardown-sandbox-clear — pairteardown clears a worktree's sandbox root before removing it
18 — treadle-callers — remove treadleengine's callers outside the package
19 — treadle-delete — delete treadle, its stencils and roles, with the agents.yaml reinterpretation
20 — crucible-docs — delete crucible and move its method into hardenshed's package doc and the docs

## Shared Decisions

### Decision: hardenshed-import-direction

- **Decision:** `internal/hardenshed` imports neither `internal/fabriccli`, `internal/configreg` nor `internal/lymcwd`.
  It takes absolute paths and told seams; `internal/hardencli` and `internal/loomcli` wire `fabriccli.CloneAndWire`, fabric commits, runproc, reed and the gate call into it.
  `internal/shedrecipe` imports `internal/hardenshed`, never the reverse, and `internal/shedadapters` learns the Hardener files only through `BouncerConfig`'s told seams.
- **Rationale:** `configreg` registers `hardener.yaml` from `hardenshed`, and `fabriccli` imports `configreg`, so either import would cycle; PATTERN-cwd-resolution keeps `lymcwd` out of engines.
- **Applies to:** cards 3, 5–11, 14, 16

### Decision: campaign-bundle

- **Decision:** One `hardenshed.Campaign` per run carries the loaded config and every seam the ring's rows need; `shedrecipe.Env.Harden` passes it, nil for every other recipe, and each Harden row refuses a nil one.
  The review and Bouncer rows opt in with the row key `harden_round: true`; the fix row is the `HardenFix` engine, whose gates are fixed.
- **Rationale:** D3 keeps the burler and Bouncer engines and adds no round runner, so the Hardener behaviour rides on the existing rows as a decorator and told seams.
- **Applies to:** cards 9–11, 13, 14

### Decision: live-gate-pass-on-cap

- **Decision:** The `live` gate entry runs with `live_attempts` attempts and `pass_on_cap`, and the gate writes the round's live report, red with its failures deferred, on every failing call.
- **Rationale:** D4 ends the row instead of halting the run once the attempts run out; a pass-on-cap entry lets the run through and the last report stays red for the facts and the auto-accept bound.
- **Applies to:** cards 6, 10, 11

### Decision: hub-gate-keys-are-a-hub-write

- **Decision:** No card writes lym's own hub's `harden-live`, `harden-live-copies` and `harden-burler-fix` values (D3, D4: `[2]`, `[2]` and `[0, 1]`).
  They are operator writes to the hub's `gate.yaml`, made outside the plan after the task lands, as the record's Added after Discussion entry "Hub gate keys are operator writes after landing" decides.
  The change description's landing section names the three writes for the operator to run from the prime:
  `lym config gate --set 'gates.harden-burler-fix={tiers: [0, 1], scope: diff}'`,
  `lym config gate --set 'gates.harden-live={tiers: [2], scope: diff}'` and
  `lym config gate --set 'gates.harden-live-copies={tiers: [2], scope: diff}'`.
  Preflight's hub-key refusal (card 5) names the two live keys' writes, so a hardener run on a hub that has not made them halts with the writes to make;
  until the operator sets `harden-burler-fix`, the `verify` gate runs the template's tier 0 alone.
- **Rationale:** the hub's `gate.yaml` lies under the hub's `_bolt`, outside the task worktree, and hub config is the operator's call;
  the shipped template keys (card 2) carry the keys, and the hub's values are its own tuning.
- **Applies to:** cards 2, 5

### Decision: harden-reap-seams

- **Decision:** `loomcli.HardenReapDeps` is the one builder of the reap's production seams; the Harden rows, Finalize's reap and `lym harden sandbox clear` all take it.
- **Rationale:** the clear verb and the run's rows must end the same processes and servers; one builder keeps the bound of D5 in one place.
- **Applies to:** cards 14–16

## PATTERN changes

- `PATTERN-child-processes` — changes: the Run process owner section admits the Hardener sandbox reaper beside `lym tender reap` as a user of `EndProcess`, bounded to processes under a `hardener` run's sandbox root that no record names (D5).
