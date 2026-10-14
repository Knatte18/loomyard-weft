---
format: 5
approved: false
language: go
---

# Plan: Perch budgets and stops: one budget shape for loom and darn, zero-round perches per run, arbiter only when needed

Loom and darn read their review budgets through one shared key family in a new `internal/reviewbudget` leaf.
Every perch, loom and darn, takes a budget of 0 as zero rounds, settable per run through `run_settings`, with no guard key; a loom perch at 0 keeps its writer → Bouncer route and the Bouncer settles at once, `skip_merge_review` is retired, and darn at 0 keeps Darn-Write → Merge-In.
The arbiter's awaiting trigger narrows to `hard-cap` and `circling` escalations through one eligibility function in a new `internal/arbiterstop` package, which the verb, the Go driver and batten share.
Batten holds an arbiter-eligible stop's notice until the arbiter has handed on, and the LLM driver stencil sends a twice-failed transient to its fork first and relays no notice after a resume.
The decision record's "Added after Discussion" entries override D2, D3, D4 and the acceptance criteria they contradict: there is no `skip_review`, no darn migration and no `PrepareRunSettings` reorder.

## Card Index

1 — reviewbudget-package — a stdlib-and-configengine leaf declaring the review budget key family, its categories, cap resolution, layer routes and bound checks
2 — zero-round-perch-routing — a perch budget of 0 stamps zero_rounds on its Bouncer, which settles at once; the merge-review route switch goes
3 — loom-config-zero-rounds — loom.yaml's budgets move onto reviewbudget with null inheriting and 0 meaning zero rounds, skip_merge_review retired and migrated
4 — darn-config-per-run-budget — darn's budget keys lose the guard, become settable per run and refuse by layer
5 — pending-escalation-cause — shedadapters reads a pending escalation's cause
6 — arbiter-eligibility-package — one eligibility function in arbiterstop; an awaiting stop is eligible only at a hard-cap or circling escalation
7 — arbiter-hold-reading — arbiterstop reads the facts batten's eligible-stop hold decides on
8 — batten-eligible-stop-hold — batten holds an eligible stop's notice until the arbiter hands on, and measures the report wait from the hold's end
9 — driver-stencil-arbiter-flow — the LLM driver sends a twice-failed transient to its fork first, lets the verb decide eligibility and relays no notice after a resume

## Shared Decisions

- A perch budget of 0 means zero rounds in every layer and in `run_settings`; a null per-perch budget takes `review_max_bounces`; hard caps keep 0 as inherit.
- The arbiter eligibility rule has one home, `internal/arbiterstop`, because `internal/battencli` must call it and cannot import `internal/loomcli` without linking tree-sitter.
- `battenshed` holds no copy of eligibility: it reads a `ChildArbiterStop` whose `Eligible` field `arbiterstop` fills.

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
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
