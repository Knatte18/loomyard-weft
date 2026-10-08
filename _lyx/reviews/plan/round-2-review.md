---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-hubreconcile.md (reconcileWorktree steps 1-4) and _lyx/plan/05-fabric-walk-reads.md (CodeWorktrees, OpenCodeWorktree)
    summary: The walk reconciles every worktree git lists, so a pair that Topology.Add has not finished wiring gets its config templates written into a real _lyx dir where Add's junction goes, and Add then refuses to wire it
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/10-batten-run-reconcile.md (Tests, second row)
    summary: The row asserting the stamp is written before any seed exists cannot observe that ordering, since arm for run auto-seeds before it returns
---

### [MEDIUM] The walk writes config into a pair that Add has not finished wiring

**Location:** `_lyx/plan/07-hubreconcile.md`, `reconcileWorktree` steps 1–4 and `ensure` step 6; `_lyx/plan/05-fabric-walk-reads.md`, `CodeWorktrees` and `OpenCodeWorktree`.

**Issue:**
Card 5's `CodeWorktrees` lists every entry `git worktree list` prints, skipping only prunable or vanished ones.
Card 7 reconciles each listed pair once `OpenCodeWorktree` succeeds and `MergeBlocked` reports false.
`Topology.Add` (`internal/fabricengine/add.go`) makes a pair visible to `git worktree list` when it creates the warp worktree.
It then creates the weft worktree, the portal and the launchers.
Only after that does it wire the warp's `_lyx` junction (`WireJunctionsWith`, step 10b).
Between the weft creation and the junction wiring, `OpenCodeWorktree` succeeds, because `newPaired` checks only that both directories exist.
In that window, `configsync.ReconcileAll(w.Anchor, …)` finds no config file under `<warp>/<anchor>/_lyx/config/`.
For an absent file, `ReconcileAll` materializes the template (`configsync.go`: "absent files materialize the template verbatim"), which creates a real `_lyx` directory in the warp.
`WireJunctionsWith` refuses a real directory where the link goes (`junction.go`: "a real directory is refused"), so the concurrent `lyx batten run`'s `Worktree-Create` row fails.
This is the case Decision 5's re-enumeration exists to handle: a pair that becomes enumerable while another start verb's walk holds the hub lock.
`Add` does not take the hub lock, so the two do not serialize.
The same applies to a pair whose `Add` was killed partway through (the case `fabricengine.PairComplete`'s doc describes): every later walk writes a real `_lyx` tree into it.
The walk also commits those files through `Fabric.Commit` even though they sit in the warp rather than the weft.
Card 7's test rows cover a deleted pair and a pair added between listings by `afterEnumerate`, but none covers a pair listed before it is complete.

**Fix:**
Make the walk skip a listed pair that is not a complete pair, using the predicate fabric already has for that question (`fabricengine.PairComplete`, or a path-based form exported beside `CodeWorktrees`).
Decide, and state in card 7, whether an incomplete pair keeps the stamp stale.
It can be stamp-neutral: a pair `lyx batten run` is creating is covered by card 10's unconditional pair call after `Add`, and a `lyx fabric add` pair is Decision 4's bounded escape.
Add a card 7 `ensure_integration_test.go` row: a pair whose junction is not yet wired is listed but left unwritten, with no `_lyx` directory created in its warp, and its later completion still wires the junction.

### [LOW] The batten stamp-before-seed assertion cannot observe its own ordering

**Location:** `_lyx/plan/10-batten-run-reconcile.md`, Tests, second row ("A second row runs `arm` for `run` against an absent stamp and asserts the stamp is written before any seed exists").

**Issue:**
`battenCLI.arm` for `run` resolves the location, refuses when needed, resolves the run id and then runs the gated auto-seed before `wire` (`internal/battencli/arm.go` header comment).
After `arm` returns, the stamp and the seed both exist, so a check made once `arm` has returned cannot tell which was written first.
As written, the assertion reduces to "the stamp exists", and it would still pass with the call moved after the seed.
Card 9 handles the same question for `loom resume` by asserting on the stamp "whatever its own exit".

**Fix:**
Restate the row as something a test can observe.
One option: make `arm` refuse after the reconcile call and before the seed, for example a run id that fails the self-address refusal, and assert that the stamp is written and no seed exists.
Another option: assert only that the stamp exists after `arm`, and drop the "before any seed exists" wording.

## Focus verdicts

Focus 1 (required `{{.edit_directive}}` marker; tests that hand-build a stencil's text or its values map): **not a defect.**
I grepped every test that hand-builds stencil text or values (`stencil.Fill`, `parentdirective.MarkerName`, `parent_directive` fixtures, hand-written `parent-directive-*` stencils).
Each card's targets name the tests in its packages that need a change:

- Card 17: `websterengine/template_test.go` values maps, plus the `claims_test.go` pin.
- Card 18: burler `template_test.go`, `bouncer_seed_test.go`'s `TestBouncer_MarkerCompleteness_BothTemplates`, and `shedrecipe/entries_bouncer_test.go`'s `minimalBouncerConfig`, which also feeds `TestBouncerEntry_ParentNamePassesToJudgePrompt`.
- Card 19: treadle `template_test.go`.
- Card 20: the three `*StencilFixture` constants and `finalizeConflictStencilFixture`.

The other hand-builders need no change:

- `burlerengine/engine_test.go` and `bouncer_fixture_test.go` only mention `stencil.Fill` in comments.
- The `withStencilOverrides` rows in the bouncer tests assert an `unknown_marker` refusal, which still holds.
- `loomengine/discussion_test.go` overwrites the discussion template without either directive marker, so the dropped value is harmless and the test asserts nothing about the directive.

Every fixture directory those renderers read is built through `stencilkit.Seed`/`SeedInto`, or through card 18's edited `minimalBouncerConfig`, so `edit-directive` is present.
Each planned "contains `Edit or Write`" assertion is reachable:

- Card 16: `TestPlanSpec_PatternDirective`.
- Card 17: the real-render parent-directive test at `template_test.go:885–911`.
- Card 18: bouncer judge and seed over `stencilkit.Seed`.
- Card 19: `judge_test.go` over `stencilkit.Seed`.
- Card 20: the three fixtures gain the marker, and the driver stencil is seeded.

Card 21's restatement scan is also safe.
The only existing stencil sentence matching its phrases is the Master line card 17 shortens (`webster-template-master.md:201`).

Focus 2 (pair-path normalization in the pair call): **not a defect.**
Card 5 sets `CodeWorktree.Path` to `filepath.Clean(filepath.FromSlash(entry.Path))`, the same form `destroy.go:697` and `remove.go:826` compare `List` entries in.
Card 7 compares it against `filepath.Clean(opts.Pair)`.
Card 10 passes `fabricengine.WorktreePath(location, slug)`, a `filepath.Join` over `HubPath`.
`HubPath` comes from `git rev-parse --show-toplevel` (`lyxcwd.resolveCore`), so it has the same real-path origin as the porcelain listing.
Card 5's `openparent_integration_test.go` row pins `Path == WorktreePath(slug)` on root-anchored and subpath-anchored hubs.
Card 7's last `ensure_integration_test.go` row drives `Options{Pair: …}` with the path built the way `lyx batten run` builds it.
