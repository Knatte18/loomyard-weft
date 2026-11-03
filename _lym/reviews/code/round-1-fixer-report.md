### Changes

- **F1** fixed as proposed.
  - `internal/configcli/configcli_integration_test.go`: a new subtest runs `reconcile --apply` from the pair and asserts a planted `newer_key` leaves the worktree `loom.yaml`.
  - The default-layer retired-keys subtest also plants `newer_loom_key` and `newer_landing_key` and asserts both are gone.
  - Dropping `PruneUnknown` at `configcli.go:560` or `hubwide.go:184` now fails a test.
- **F2** fixed as proposed.
  - `internal/hubreconcile/ensure_integration_test.go`: `runStaleBuildReconcile` commits a `newer_worktree_key` into the pair's `logger.yaml` and asserts it survives `Ensure`.
    The proposal's wording named "a pair worktree's sparse config file" without a module; `logger` is per-worktree and not seed-only.
  - `internal/fabriccli/hubmutation_integration_test.go`: a new step `ReconcileKeepsAKeyTheTemplateLacks` commits a `newer_board_key` into the board's `board.yaml`, runs `fabric reconcile` and asserts it is kept.
    It commits the key because a dirty board makes the hub-wide heal skip (self-check finding).
  - The optional clone row is not added (see Deferred).
- **F3** fixed differently.
  - Proposed change not made: run `yamlengine.Prune` over `withoutRetired` in `pruneSparse` and return its pruned bytes.
    `Prune` removes every unit the template lacks, so those bytes would drop the very keys `pruneSparse` preserves when `PruneUnknown` is unset (`yamlengine/layers.go` `pruneMapping`: a key with no template value is appended to `removed` and skipped from `retained`).
  - Own change: `configengine.removeKey` now also removes a mapping the removal empties, so `DropRetired` returns the bytes a plain prune would, and the emptied mapping is never listed as preserved.
    `pruneSparse` is unchanged.
  - `TestDropRetired` in `internal/configengine/layers_test.go` pins the nested retired key beside a sibling, as the only child, at top level, and a document with no retired key.
    The "only child" row fails without the change.
  - No registered module declares a nested retired key, so the case is unreachable today; the row pins it for the day one does.
- **F4** fixed as proposed: `pattern/PATTERN-config-layers.md` and the `hubreconcile/doc.go` paragraph now say the key is carried in `Result.Preserved` and logged when the walk writes the file.
- **F5** fixed as proposed: `PATTERN.md` and `pattern/PATTERN-config-layers.md` say "hub base or worktree key", and the `ReconcileAll` comment says the standalone worktree base is fully reconciled whatever `PruneUnknown` says.
  The `configsync.go` package header carried the same broad wording and is narrowed as well (self-check finding).

The gate (`lym gate test --tier 1` over `./internal/configcli`, `./internal/hubreconcile`, `./internal/fabriccli`, and tier 0 over those and `./internal/configengine`, `./internal/configsync`) passes.
Commits: one per finding, and one for the self-check fixes.

## Self-check

- fix-code-correctness, MEDIUM: the `ReconcileKeepsAKeyTheTemplateLacks` step could pass with the defect in place, since an uncommitted key makes the heal skip. Fixed as proposed: the key is committed to the board and the original committed back.
- fix-consistency, LOW: `configsync.go` package header still said "a base or worktree file". Fixed as proposed.
- fix-consistency, LOW: `removeKey` doc comment omitted the emptied-parent removal. Fixed as proposed.
- fix-consistency, LOW: `runStaleBuildReconcile` doc comment omitted the new assertions. Fixed as proposed.
- fix-affected-set, LOW: `clone.go:58` and `clone.go:122` are untested automatic callers. Deferred, same reason as the optional clone row below.
- fix-proposal: no findings.

## Disputed

None.

## Deferred

- Clone call sites (`clone.go:58`, `:122`) are not pinned by a test.
  The review marks a clone row optional, and a clone fixture needs a full `CloneAndWire` hub that no existing test in these packages builds for reconcile assertions; the walk and `lym fabric reconcile` rows cover the same flag's absence on the sibling automatic callers.
