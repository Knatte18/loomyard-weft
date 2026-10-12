# Layer module config and carry run settings on board tasks

Every module's config now resolves through a stack of layers, key by key, and a board task can carry run settings that batten applies to its pair without a tracked config change.

## What changes

A per-worktree module resolves as template, repo default, worktree file, local override.
A hub-wide module resolves as template, hub file, then the reading worktree's local override.
Standalone, with no board dir, the stack is template, worktree file, local override.

- The repo default of a per-worktree module sits at `<BoardDir>/_lyx/config/<module>.yaml`, beside the hub-wide files, and is committed on Bolt like a hub-wide edit.
- The local override sits at `<anchor>/.lyx/config/<module>.local.yaml`, untracked.
  Its `.local.yaml` suffix keeps it apart from the editor staging copy.
- A key missing from a present file falls back to the layer below and finally to the template, which closes the gap where a pair created before a new key was added ran with the Go zero value.
- `lyx config <module>` gains `--layer default|worktree|local`.
  `--resolved` prints the merged result with the layer each leaf came from, and `lyx config reconcile` reports one result per module and layer.
- `boardengine.Task` gains `run_settings`, an object of module name to that module's keys.
  `lyx board upsert` runs each module's `Migrate` hook over it and validates it through `internal/configreg`.
  Batten's Seed-Child row writes one local override per named module into the pair and removes the local files of the modules it does not name.
- The orch stencil's hand step for a small task's `review_max_bounces` is replaced by `run_settings` `{"loom": {"review_max_bounces": 2}}` on the board entry.
  Landing no longer reports a per-worktree config change for it.

Most of the code change is the module loaders (`reed`, `shuttle`, `orch`, `webster`, `loom`, `darn`, `landing`, `gate`, `board`, `fabric`, `batcher`, `burler`, `models`, the logger) moving onto the layered `configengine` load, with the stacking primitives in `internal/yamlengine/layers.go`.

## Design decisions

- **Merge rule.**
  Mappings merge at every depth; scalars and lists replace whole.
  An open map, and each top-level alias in `models`, is carried whole from the highest layer that holds it.
  No layer can delete a key from a layer below.
- **One expansion.**
  Layers merge unexpanded and the merged document is expanded once, against the reading worktree's `.env` for a per-worktree module and the board dir's for a hub-wide one.
- **Base full, upper sparse.**
  Reconcile keeps the base layer full and keeps every layer above it sparse: it prunes retired keys and runs `Migrate` there but never adds a key.
  Seed-only modules (`models`, `burler`) keep the operator's own aliases, lenses and fans untouched in every layer.
- **Strictness.**
  `Load` and `LoadOrTemplate` keep their policies for the base layer.
  An absent upper layer, or an empty or comment-only local file, is no error.
  An unparseable layer file, or a local file naming a key its template lacks, returns `ErrInvalid` naming the file and key.
  `models` aliases and open-map entries are exempt from the unknown-key refusal.
- **The local layer is bounded.**
  This narrows the board entry's "any key of any module".
  `configreg.Module` gains `LocalKeys` for hub-wide modules and `LocalDenied` for per-worktree ones; a local file or `run_settings` naming a closed key is refused at load or at board write.
  The lists were derived by testing every template key against these categories: runs an executable, sources a file, reaches across pairs (including a shared process's boot option or what lands on the parent branch), or switches off or hollows a guard such as a review or approval gate.
  A test pins that every template key is classified.
  So `darn.verify`, `darn.review_max_bounces`, `landing.publish_verify`, `landing.require_pr_to_base`, `landing.squash`, `landing.co_authored_by`, `reed.shell`, `shuttle.claude`, `loom.parent_review_wait_min`, `loom.discussion_producer` and `burler.lenses` cannot be set locally.
  What a local file can change is session tuning: timeouts, budgets, model picks and colors.
  The new `PATTERN-config-layers` entry states the rule, and `PATTERN-hub-wide-config`, `PATTERN-config-strictness` and `PATTERN-durable-vs-ephemeral-state` are updated to match.
- **One-time migration.**
  The hub reconcile walk seeds each per-worktree module's absent repo default from the prime, then strips from each worktree lacking a tracked marker file every key equal to what the layers below resolve to, in that worktree's existing commit.
  The comparison is over unexpanded values, so a `${env:...}` marker equals only the same marker text.
  The walk lists `stripped` and `inherited` keys per worktree and module.
  A key the prime's file holds beyond the template, such as a seed-only alias, can now resolve to the prime's value in a worktree whose file lacked it; `inherited` lists those.
  An interrupted walk, or one that skipped a pair, is completed by the next walk.
- **`run_settings` validation.**
  A key must exist in the template, sit under an open map or under a `models` alias, match the template's shape, and be open to the local layer.
  A `run_settings` write from a slug-named session is refused through the same guard as the hub-wide config write.
  A change on a task already past Seed-Child returns a notice that it applies to the next run.
  `lyx board intake` never sets it.
- **Seed-Child.**
  It reads the task fresh, writes atomically before the child's seed, and is idempotent on resume.
  A key that does not validate routes it to Stuck with the way forward named, and the refusal has a row in `contracts/specs/refusal-spec.md`.

## Open risks

- The migration commits a config change in every running pair, so a pair whose overrides differ from the prime's sees one landing warning after it.
- An agent in a pair can still write its own local file, and nothing at landing shows it; the pair's git-ignored `.env` was already such a route for marker-carrying keys.
- `burler-split-budget` adds per-perch loom keys; a run setting naming one validates only once that task lands.
- `orch-wake-policy` also changes batten; whichever lands second rebases onto the other.

## To check by hand

- Clone-walk output tests are not planned for the clone walk added in the migration (`clone-output-tests-unplanned-after-card-15`, BLOCKING when raised): check that `internal/hubreconcile` and `internal/hubforge` have tests pinning what the clone walk writes.
  The last fix, `forgetCloneCorrespondence` in `internal/hubforge/hub.go`, rewords the weft prime's reconcile commit so fixtures start with no warp-to-weft correspondence; no test pins that reword.
- `hubreconcile-test-edits-not-member-glyphs`: check the `internal/hubreconcile` test edits against the plan's declared file set.
- `reseed-case-collides-with-newstalehub-migration`: check that the reseed case and the `newstalehub` migration fixture in `internal/hubreconcile` tests do not shadow each other.
- Several commits edited files with `sed -i` or shell heredocs instead of the edit tools; the results passed their gates, but skim the diff for unintended rewrites.
- The final fix ran `lyx gate test --tags integration ./...` clean, but the default-tag tier was not rerun after it; run `lyx gate test ./...` before pushing.
