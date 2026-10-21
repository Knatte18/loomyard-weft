---
format: 5
approved: false
language: go
---

# Plan: tuned-defaults-template

The loom setup tuned in the loomyard hub becomes lyx's embedded template, and every hub's base layer becomes sparse over the template, so a later template change reaches every hub with no migration.
Cards 1 to 3 move D1's values into the templates.
Card 4 teaches `configengine` the sparse hub base: its header, the board marker's path, the sparse `--set` and editor scaffold, and the unlogged fill.
Cards 5 to 7 make `configsync` keep a hub base sparse: the former-values table and the worktree's former-value drop, the sparse seed, prune and one-time strip of `ReconcileHubWideAt`, and sparse key moves.
Card 8 runs the strip in `hubreconcile`'s walk and commits the marker with it; card 9 carries the operator's addition, a failed reconcile that warns and goes on; cards 10 and 11 bring `lyx config` and `lyx fabric clone` along.

## Card Index

1 — loom-tuned-defaults — the loom template takes D1's tuned review models, fans, lenses, budgets, accept and driver
2 — shed-darn-tuned-defaults — the shed and darn templates take `driver_kind: go` and darn's tuned models and fans
3 — claude-orch-webster-batcher-tuned-defaults — the claude, orch, webster and batcher templates take D1's values, orch's floors with them
4 — configengine-sparse-hub-base — `Spec.SeedOnly`, the header and board marker, sparse hub `Set` and `Edit`, no fill log for a hub base
5 — former-template-values — the former-values table, and an unmarked worktree's strip drops former values
6 — sparse-hub-base-reconcile — `ReconcileHubWideAt` seeds sparse, never fills, strips once under `StripBase` and writes the marker
7 — sparse-key-moves — key moves into or out of a hub base write it sparse and keep it present
8 — walk-strips-base-once — the walk strips the base once, commits the marker, and an absent marker makes the stamp stale
9 — reconcile-failure-warns — a failed stamp-gated reconcile warns with its way forward and the start verb goes on
10 — config-cli-sparse-hub-base — `lyx config`'s hub-wide editor scaffolds the header and its docs state the sparse base
11 — clone-writes-base-marker — `lyx fabric clone` marks a board it seeded from nothing in its own board commit

## Shared Decisions

- The added-after-Discussion decision is carried this way: the stamp of the template version is `hubreconcile`'s build stamp, whose key folds `configreg.Fingerprint`, which covers every template's text, so no second stamp is added; "when lyx reads a hub's config" is the existing stamp-gated `Ensure` before any module config loads, at the start verbs and the Preflight row, which keeps `PATTERN-binary-change-reconcile`'s single route; "drops seeded default-layer entries equal to the old template" is card 8's walk strip, gated by the board marker as D3 decides, since after it no base holds a seeded entry; the new behavior is the failure half, card 9's `EnsureOrWarn`, and "the command to reconcile by hand" is re-running a start verb, whose walk is the reconcile, after fixing the file the error names.
- A seed-only module is told to `configengine` through a new `Spec.SeedOnly` field, set in `modelspec` and `runprofile` and pinned equal to `configreg.Module.SeedOnly` by the registry test, because `Set`, `Edit` and `load` take a `Spec` and must leave a seed-only base full (D2's Out list) and keep its fill log (D5).
- The board marker is `<BoardDir>/_lyx/config/base-sparse`, declared in `configengine` beside the worktree `sparse` marker, and the header-only base is `configengine.SparseBaseHeader`.
- The former values are a module-to-unit table of YAML value texts in `configsync`, rendered per module as one YAML document so every comparison with them is `yamlengine.Strip` with the module's open maps, the comparison the worktree strip already makes.
- "If the reconcile fails, the hub's config is left as it was" is read per failing step: the walk restores the failing step's files, the board's with its marker or one worktree's, while steps that committed before it keep their commits, as D3 and `internal/hubreconcile/doc.go` already decide; a committed step is a finished reconcile of its part, never a half-written config.
- `orchengine`'s `defaultThresholdTokens` and `defaultSoftThresholdTokens` follow the template to 200000 and 150000, beyond D1's template lines, because their const block defines them as the template defaults a non-positive value floors back to, so keeping the old numbers would make that definition false.
- The scaffold of an absent base is one function, `configengine.BaseScaffold`, used by `configengine.Edit` and `lyx config`'s hub-wide editor alike.
- `ReconcileHubWideAt`'s strip is opt-in through `HubWideOptions.StripBase`, set only by the walk, mirroring the worktree strip behind `Options.Migrate`.

## PATTERN changes

- `PATTERN-sparse-hub-base` — adds: Writing a hub's base file (a hub-wide module's hub file or a per-worktree repo default, not seed-only): only the units the hub tuned, never a template fill; only `hubreconcile`'s one-time walk strip drops a unit the template holds, bar `lenses`' template-equal entries.

## verify:

```bash
go build ./...
lyx gate test --tags integration ./...
```
