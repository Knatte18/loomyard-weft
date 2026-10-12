# Fixer report — plan, round 2

Every finding's premise held against the code.
F2–F6 are fixed as proposed, taking one of the offered options where the fix key offered a choice; F1 is fixed with two own changes inside the proposed shape.

## Changes

- **F1** (cards 9, 15, overview), fixed differently in two details.
  The ordering window is closed by the fix key's second option: card 9 keeps loomcli's and webstercli's loom callers on the transitional `configengine.Dirs{Anchor: location.AnchorPath()}` (its `Uses:` drops `fabricengine#ConfigDirs`), and card 15 switches them to `fabricengine.ConfigDirs(location)`, adding `internal/loomcli/arm.go`, `internal/loomcli/wiring.go` and `internal/webstercli/wiring.go` to its `Edit:`.
  The overview's "transitional loader geometry" decision states the loom exception and applies to card 15 too.
  Card 15's `CloneAndWire` ends by calling `hubreconcile.Ensure(..., hubreconcile.Options{})`, never passes `SeedDefaults: true`, and its Intent states that bound; `internal/hubforge/doc.go` is in `Edit:` with its seeding contract corrected.
  Own change 1: the geometry is built as `hubreconcile.Geometry{BoardDir: res.BoardDir, WorktreePath: l.WorktreePath()}` from values `CloneAndWire` already holds, not through `hubgeom.ReconcileGeometry`, which would pull hubgeom's reed, burler and preflight imports into fabriccli for a check (`preflight.BoardLyxPresent`) a fresh clone always passes.
  Own change 2: the walk writes the build stamp, and several hubforge-based tests start from a hub with no stamp (`newStaleHub` in `internal/hubreconcile/ensure_integration_test.go:145`, `internal/loomcli/smoke_prebootstrap_integration_test.go:198`, `internal/preflightshed/preflight_integration_test.go:162`).
  So `hubforge.cloneHub` removes the stamp after `CloneAndWire`, keeping the seeded defaults; `internal/hubforge#NewHub`, `#cloneHub` and `#SeedConfig` join `Edit:` for that and their stale docs.
  Card 15 also has `CloneAndWire` record each seeded default as `KindFileWritten`, so the clone's mutation record stays complete.
  `TestRunCLI_CleanHubScenario` asserts a fresh clone's board carries the defaults; card 21's integration row now finds a repo default in its hubforge hub, with no change to card 21.
- **F2** (cards 2, 15): the fix key's second option.
  Card 15's `Ensure` counts a stamp naming the running build as stale when any per-worktree repo default is absent, adding `internal/hubreconcile#ensure` and `internal/hubreconcile/doc.go` to `Edit:` and a case in `TestEnsure_StaleBuildReconcilesAndCommitsEveryWorktree`.
  Card 2's strict-load refusal for an absent board-dir base splits by module kind: `run "lyx fabric reconcile"` for a hub-wide hub file, and re-running the start verb, as `lyx loom resume`, whose hub reconcile seeds the default, for a per-worktree repo default, both with the refused-session clause.
  `TestLoad_Refusals` gains both rows.
- **F3** (cards 2, 7, 8): card 2 creates `configengine.Stack` and `StackLayers(dirs, spec) (Stack, error)`: `Doc` merges the present layers through `yamlengine.Merge` with no template fill, `Layers` keeps the per-file docs, and `Origins` maps each top-level key to the highest layer holding it.
  `Load` and `LoadOrTemplate` merge through it; `TestLayers` gains its rows.
  Card 8's `burlerengine.LoadConfig` validates each layer and decodes the one `Doc` (burler has no open maps, so lenses and fans merge per name), and `modelspec.LoadRegistry` overlays the one `Doc` on `builtins()`; neither keeps a merge loop.
  Card 7's retired-key attribution reads `Origins["profiles"]`; both cards' `Uses:` name `StackLayers`.
- **F4** (cards 4, 5, 9, 10, 13): card 13's `TestModules_LocalLayerRefusals` keeps only `darn.review_max_bounces`, `landing.require_pr_to_base`, `reed.mouse`, `loom.discussion_producer` and `loom.plan_fan` refused, and `shuttle.claude_prompt_cache_ttl_roles.<role>` passing.
  Each loader card's sentence now says card 13 owns the keys no loader card pins and names the keys it does not repeat.
- **F5** (card 16): `internal/hubreconcile#snapshotConfig` joins `Edit:`; it records `configengine.SparseMarker(anchor)` as absent (nil) when not on disk, so `restoreConfig` removes a marker a failed walk wrote.
  The `TestEnsure_MergeBeginningBeforeCommitRestoresTheFiles` assertion now says no marker is on disk after the restore.
- **F6** (card 2): only template fills are logged, because a key inherited through a sparse layer is the normal case in a hub and `--resolved` (card 18) shows its layer; `TestLoad_LayerStack` pins a template fill logged with `template` and an inherited key logging nothing.

## Disputed

None.

## Deferred

None.
