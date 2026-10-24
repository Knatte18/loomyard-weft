---
format: 5
approved: true
language: go
---

# Plan: dotnet-scoped-verify

Verify gates in a .NET hub build once and test only what a card, batch or diff touches, keeping the whole solution to the landing gates.
The cards add the `gate.yaml` keys first, then the build step's lines and records, the plan parser's C# card seeds and the `internal/dotnetimpact` derivation, and then wire `{filter}`, the `dotnet` scoper, the fingerprint and `carry: false`, `inert_paths` and the shared build step into `lyx gate test`, closing with a real `dotnet` run and `docs/dotnet-hub.md`.

## Card Index

1 — gate-config-keys — load scoper dotnet, the tier keys filter, shared_build and carry, the build step and inert_paths, with their refusals and the inert matcher
2 — build-run-lines — the build summary and timeout lines, the build run record and the landing failure record's build flag
3 — card-csharp-seeds — planparser's per-target C# seeds for the dotnet scoper's --card form
4 — csharp-source-scan — internal/dotnetimpact's C# lexer and per-file source model
5 — test-class-selection — the type-name closure, test classes, project grouping and filter terms
6 — impactset-diff-reads — impactset's exported diff reads ChangedPaths and AncestorBase
7 — derive-entry-points — the diff and card entry points with their fallbacks
8 — filter-placeholder — render {filter} from the tier filter and an invocation's terms
9 — dotnet-scoper — wire the dotnet scoper into computeScope and the per-project invocations
10 — fingerprint-and-carry — the new keys in the tier fingerprint, and carry: false
11 — inert-paths — inert-only diffs run no tier, and whole gates carry over inert-only history
12 — shared-build-step — run the build once per call inside the first opted-in tier's slots
13 — dotnet-hub-end-to-end — a real dotnet scoped run and docs/dotnet-hub.md

## Shared Decisions

- The inert matcher lives on `gateslot.Config` (card 1), so `gatecli` and `dotnetimpact` read one rule; `dotnetimpact` is told it as a predicate and imports no config.
- `Tier.NoCarry` is the negative form of `carry`, so a hand-built `Tier` keeps today's carrying behaviour at its zero value.
- The build timeout line has its own prefix, and `gateslot.FindTierTimeout` finds it as tier `build`, so `verifytree` reads it as timed out without a change of its own.
