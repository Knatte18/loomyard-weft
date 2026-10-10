---
format: 5
approved: true
language: go
---

# Plan: gate-test-scope

Every gate runs the test tier its position needs and no more, and the hub as a whole never runs more gate builds and tests at once than its configured slot count allows.
The plan first builds the hub-wide slot engine and its `gate` config, threads the slot through every Go-side gate run (`verifytree.Verify`, the card-verify rerun, the plan gate's type load), and adds `lyx gate test`, the one slotted route for an agent's module-wide run, with a standing agent deny and a plan-gate check against raw module-wide `go` runs.
It then narrows the card gate to tier 1 of the card's own packages, adds a once-per-batch tier 2 gate, moves `landing` hub-wide behind a `lyx config` refusal for pairs, persists a Publish failure record that the round gate and the Webster-Review rubric read, tightens the comment lint's comma-plus-conjunction rule, and records the gate model in PATTERN.

## Card Index

1 — gateslot-pool — the hub-wide slot pool engine: file-lock slots, holder records, inheritance, GOFLAGS cap, wait records
2 — gate-config — the hub-wide gate config module (slots, go_parallel, cli_wait_sec) registered in configreg
3 — verifyrun-env — verifyrun.Run takes the shell's environment
4 — verifytree-slot — verifytree.Verify acquires a slot, marks the wait, and starts its timeout once held
5 — hubgeom-gate-slots — the hub pool constructor and webster geometry's GateSlots field
6 — webster-sites-slotted — webster's gate, lyx webster verify and the card-verify rerun take the slot
7 — landing-slotted — Publish and Finalize verifies take the slot
8 — round-gate-slotted — the Webster-Burler round gate takes the slot
9 — plan-gate-slotted — the plan gate's caller-uncovered go list takes the slot
10 — gate-cli — the lyx gate test module
11 — status-gate-waits — lyx loom status shows gate-slot waits and holders
12 — raw-go-deny — standing agent deny against raw module-wide and tmux/llm-tier go runs
13 — verify-module-wide-check — the plan gate refuses a module-wide card Verify
14 — card-gate-tier1 — the card gate runs lyx gate test over the card's own packages, one step per line
15 — batch-gate — a once-per-batch integration gate and the fork's run rules
16 — gate-route-stencils — verify-fix stencil and CLAUDE.md push rule name lyx gate test
17 — config-hubwide-write-refusal — lyx config refuses hub-wide writes from pairs and pair-spawned sessions
18 — landing-hubwide — landing becomes hub-wide, seeded from the prime
19 — publish-failure-record — the Publish failure record codec in verifytree
20 — publish-writes-failure — Publish writes and clears the failure record
21 — round-gate-tier3 — the round gate runs the failing tests and a tmux pass while a record is present
22 — webster-review-failure — the Webster-Review rubric renders the checked failure record
23 — comment-lint-clause — comma-plus-conjunction breaks pass only before a clause
24 — gate-tier-docs — PATTERN records the gate table, the llm decision and the slot entry

## Shared Decisions

### Decision: nil pool means unslotted

- **Decision:** every Go-side site takes a `*gateslot.Pool`; nil runs unslotted with the parent environment, the form of a standalone run with no hub and of a unit test.
- **Rationale:** one optional parameter keeps standalone webster and tier-1 tests free of hub geometry while every hub-mode wiring passes `hubgeom.GateSlots`.
- **Applies to:** cards 4–9

### Decision: the slot is read at acquire

- **Decision:** a pool's limits are a closure that loads the board dir's `gate.yaml` at every acquire; no site captures the count at wiring time.
- **Rationale:** D5 reads the count at each acquire, so lowering it takes effect on the next acquire while a running holder finishes.
- **Applies to:** cards 1, 2, 5

### Decision: child lifetime code stays in gatecli

- **Decision:** `lyx gate test`'s process-group, `Pdeathsig` and job-object code lives in new `internal/gatecli` files only, and no card touches `internal/proc` or `internal/verifyrun`'s process-group files.
- **Rationale:** the parallel run git-robustness changes those paths; keeping D7's child code apart lets the branches merge without conflict (decision "lyx gate child lifetime stays in internal/gatecli").
- **Applies to:** cards 3, 10

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
