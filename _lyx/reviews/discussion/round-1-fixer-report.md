# Fixer report — discussion round 1

## Changes

- F1, fixed as proposed: D4 now says the Publish seam classifies the task diff at each Publish call, after the moved-parent check, against the merge base Merge-In's merge left.
  D4's Bound says the skip covers only the diff classified at that call.
  A new criterion covers a PR-Gate rework that adds a production path.
- F2, fixed differently: D1 declares `tree_scope` an open map (`ConfigOpenMaps`).
  The last criterion now names one `--set` command that turns scope on, and one that turns it off.
  Not made: the proposal to turn scope off with "one call carrying both keys' `--set` pairs".
  `yamlengine` says `SetValues` replaces P whole only "when P holds a scalar or a list, not a mapping", so `--set` cannot empty a `tree_scope` that holds trees.
  Instead, the half-set refusal is narrowed to a `tree_scope_verify` with no tree.
  A lone `tree_scope` is accepted as scope off, so `--set tree_scope_verify=` alone turns scope off, and a tree is removed through `lyx config darn`'s editor.
  The refusal criterion and the open-map `--set` and reconcile criterion were updated to match.
- F3, fixed as proposed: `tree_scope_verify` is `CategoryExecutable` and `tree_scope` is `CategoryGuard`.
- F4, fixed as proposed: D1 refuses a tree key holding a `.` or `..` segment, which now covers the old `.` refusal.
  It also refuses two keys that are equal after trimming.
  Each refusal has its own criterion.
- F5, fixed as proposed: two criteria were added.
  One covers a tree-scoped arrival with a `publish_verify` record and no `tmux` pass.
  The other covers a Merge-In resolution that flips the diff to production.
- F6, fixed differently: the Scope In bullet names loom's round-form verify gate at both Webster-Burler-Fix and Merge-Burler-Fix.
  The Out bullet and its rationale name only the `impactset`-derived command as changed.
  Not made: naming only Webster-Burler-Fix, as the proposal did.
  `contracts/recipes/loom-recipe.yaml:575` makes Merge-Burler-Fix's gate run "as Webster-Burler-Fix's does", so "nothing else of loom" would have been false.
- F7, fixed differently: D5 states the scan directories and refuses a scan error before any slot, naming the file and line.
  Not made: the single way forward "move the marker".
  `internal/impactset/guards.go:75,106,110` raise three kinds of error: a parse error, a misplaced marker, and a marker in a file that no integration-tier run compiles.
  D5 names the way forward for each kind.
  A criterion was added.
- F8, fixed as proposed: D4's Rationale no longer says "once" and names the writer-gate fallback.

## Self-check

- fix-claims LOW: "rolls back a write that leaves `LoadConfig` failing" overstated `setChecked`. Fixed as proposed: it now says "fails with a new error".
- fix-consistency MEDIUM: the malformed and duplicate tree-key refusals named `--set`, which cannot remove a key. Merged with fix-way-forward finding 1 and fixed as proposed: those refusals now name `lyx config darn`'s editor.
- fix-consistency LOW: the Out rationale still called loom's fix-round gate scoped with no qualification. Fixed as proposed: it now names the derived command as the one change.
- fix-consistency LOW: the support log's Turn 6 and Turn 7 contradict F1 and F3. Fixed as proposed: a Round 1 entry under Review rounds records both corrections, and the turn records stay as history.
- fix-way-forward MEDIUM: the same finding as the fix-consistency MEDIUM above, merged there.
- fix-way-forward LOW: setting the command first gets a rollback that does not name the working order. Fixed as proposed: the no-tree refusal now names `--set tree_scope.<tree>=<target>` in the same call.
- fix-proposal MEDIUM: F6's "nothing else of loom" ignored Merge-Burler-Fix. Fixed as proposed: see F6 above.

## Disputed

None.

## Deferred

None.
