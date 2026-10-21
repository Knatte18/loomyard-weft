---
format: 5
approved: false
language: go
---

# Plan: test tiers

Every test run in a hub, an agent's own and every verify gate's, apart from the non-tier slot holders, goes through `lyx gate test`, which runs numbered tiers the hub declares in `gate.yaml` over a scope lyx computes, takes the slots each tier asks for, bounds each tier by its timeout, counts its tests and logs every tier run under the board.
The plan builds the config, the slot drain and the run log in `internal/gateslot`, the `go` scoper in `internal/impactset`, the tier verb and its scopes in `internal/gatecli`, then moves every verify site, prompt and guard onto rendered gate calls and retires the hand-spelled inputs: `go_parallel`, `publish_verify`, darn's `verify`, `verify_tiers` and the plan's `## verify:` section.

## Card Index

1 — open-map-flow-values — let `--set` write a whole open-map entry as a YAML flow mapping or list
2 — gate-tier-config — declare tiers, gate lists, scoper, guard tier and `refuse_raw` in `gate.yaml`, with load and use-time refusals
3 — gate-slot-drain — multi-slot acquire with an ordered drain, the holder's tier and the inheritance-only lease environment
4 — gate-run-log — the per-tier run record and log under the board, and the timeout and summary lines
5 — impactset-tree-targets — the `go` scoper: impacted targets per module over the working tree
6 — planparser-card-packages — the card-scope package rule moves into planparser
7 — gate-counters — the `go-json` and `vstest` counters
8 — gate-tier-verb — `lyx gate test` runs tiers over explicit targets or `--all`, slotted, bounded, counted and logged
9 — gate-scopes — the computed, `--base` and `--card` scopes, per-module runs, the empty set and the guard step
10 — round-gate-tiers — the round and darn writer gates run the `round` gate; the failure record names its tier
11 — card-verify-runner — the shared card `**Verify:**` runner, `lyx gate card-verify` and `HolderEnv`
12 — verifytree-unslotted — `verifytree.Verify` drops its slot and timeout
13 — webster-final-gate — webster's plan-level gate and `lyx webster verify` run the `final` gate
14 — landing-gates — Merge-In, Finalize and Publish run `landing` and `publish`; `publish_verify` and darn `verify` retire
15 — webster-prompt-gates — webster prompts render `--card` gate calls and `lyx gate card-verify`
16 — retire-verify-tiers — prompts render the hub's tiers from `gate.yaml`; `verify_tiers` and the plan verify section retire
17 — refuse-raw-guard — the shell guard refuses the hub's `refuse_raw` commands
18 — plan-gate-verify-checks — the plan gate judges `**Verify:**` lines against `refuse_raw` and `--wait`
19 — non-tier-holders — non-tier holders set their own flags, `go_parallel` retires, PATTERN and the push rule move

## Shared Decisions

- The per-tier `parallel` machine override is the sibling open map `tier_parallel`, not a path under `tiers` (card 2), because an open map is one unit in the layer merge and a machine layer holding part of `tiers` would replace the hub's tiers whole; D1's rule that each tier's `parallel` takes this machine's override holds through it, and no change to `configengine` is needed.
- A tier's prompt rendering replaces the `verify_tiers` marker with a `gate_tiers` marker filled from `gateslot.Config.TiersValue()` (cards 2 and 16).
- A `**Verify:**` value is the text on its label line (`internal/planparser/parse.go`), so the runner and the plan gate read each card's value as one line (cards 11 and 18).
- loomshed's round-gate integration test runs its derived command through the `lyx` built from the tree, which refuses `--tags` from card 8 on, so the card that rewrites both (card 10) follows the tier verb and its scopes directly, and no card between them targets `internal/loomshed`.
- The gate commands this run's own card and batch gates spell are run by the deployed `lyx`; only a test that builds `lyx` from the tree, as the round-gate and webster gate integration tests do, meets an intermediate card's verb.
- Every refusal that names `lyx config gate` carries the role clause `withConfigWayForward` already carries: a session that is not the prime takes none of these steps, reports status: FAILED and the orch runs them (cards 2 and 8), since the sessions that meet a gate refusal are mostly forks and writers.
- Outside every hub, `gateslot.LoadConfig` with no worktree `gate.yaml` names `lyx config gate`, which writes the worktree's own file there, in place of `lyx fabric reconcile` (card 2), so every standalone reader inherits the right way forward.
- `PATTERN.md` already sits above its headroom line, so the card that adds the `PATTERN-test-tiers` entry pays for it by moving two entries' detail into background files, keeping every rule whole (card 19).

## PATTERN changes

- `PATTERN-gate-slot` — changes: Running a gate build or test: only holding `internal/gateslot` slots, either inside a `lyx gate test` tier run, holding the slots its tier asks for, with the tier's config setting its parallelism and environment, or in one of the non-tier holders `internal/gateslot`'s doc lists, holding one slot and setting its own `-trimpath` Go flag, plus `-p` from `tier0`'s parallel when the hub declares `tier0`.
- `PATTERN-test-tiers` — adds: Running a hub's tests: only `lyx gate test` over `gate.yaml`'s tiers, apart from a card `**Verify:**` line and `cmd/testtiming`'s runs in every mode; a site or stencil names a gate's tier list and scope form, never a test command or target list.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
lyx gate test --tags integration ./...
```
