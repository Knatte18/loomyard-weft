# Card 2 — verbs-regenerate

**Edit:**
- `internal/fabricengine#CloneHub`
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#Topology.Prune`
- `internal/fabricengine#Topology.Checkout`
- `internal/fabricengine#Topology.Reconcile`
- `internal/fabricengine_test#removeCase`
- `internal/fabricengine_test#pruneCase`
- `internal/fabricengine_test#checkoutCase`
- `internal/fabricengine_test#reconcileCase`
- `internal/ideengine#checkTaskSlugOpensBareFolder`

**Create:**
- `plan:internal/fabricengine#regenerateHubWorkspaceLogged` -> `func regenerateHubWorkspaceLogged(rec *Mutations, l *lyxcwd.Location)`
- `plan:internal/fabricengine_test#hubWorkspacePermittedRoot` -> `func hubWorkspacePermittedRoot() string`
- `plan:internal/fabricengine_test#TestHubWorkspace_TracksTopologyVerbs` -> `func TestHubWorkspace_TracksTopologyVerbs(t *testing.T)`

**Uses:**
- `internal/fabricengine#HubWorkspacePath`
- `internal/fabricengine#SetAddFailAfterWarpHookForTest`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`
- `internal/testkit/logcapture#Capture`

**Intent:**
Make every fabric verb that creates, removes or switches a worktree regenerate the hub workspace file (D4), so a finished pair drops out of it and a fresh hub holds it at once.

`regenerateHubWorkspaceLogged(rec, l)` calls `RegenerateHubWorkspace(rec, l)` and, on an error, logs one Warn naming the error and saying to run `lyx fabric reconcile` once the cause is fixed.
It returns nothing: a regeneration failure never fails, rolls back or changes the verb's result, and is not carried in it.
Bound: what a failure lets through is a stale folder list or a truncated file, which the next regenerating verb or prime spawn rewrites; a persistent cause keeps the Warn repeating until the operator fixes it.

Call sites, each appending to the verb's own recorder:
- `CloneHub`: on its success path, after the prime layout `l` resolves and before it returns, so a fresh hub lists the prime and `Bolt`.
  A failed clone tears the hub down and regenerates nothing.
- `Topology.add` (`Add` and `AddLocal`): a flag set once the pre-flight refusals have passed (where `disableStoreAutoGC` runs) arms a deferred call, so every later exit regenerates, an `addFailAfterWarpHook` failure and every `rollbackAdd` exit included, leaving the file matching the worktrees on disk.
- `Topology.Remove`: armed once the teardown lock is held; the deferred call runs after `releaseTeardown`, on success and on every failure past that point, a refused-midway or partial `Remove` included.
  A `RemoveRefusal`, `inspectPair` or teardown-lock refusal returns before it and regenerates nothing.
- `Topology.Prune`: with `apply`, armed after the worktree listing; a dry run never regenerates.
- `Topology.Checkout`: armed once both original branches are captured, before the warp switch, so a switch, a rollback and the index-refresh failure all regenerate and a branch switch in the prime renames the first folder at once.
- `Topology.Reconcile`: once, after the pair loop and the warp-binding backfill, as the repair verb the Warn names.
`Cleanup` removes no worktree and is unchanged.
Each verb's doc comment states that it regenerates the hub workspace file and that a failure there is only logged.

Forced test edits.
The live-state matrix's `Proceeds` cells of `Remove`, `Prune` with apply and `Checkout` now change the workspace file's content, so `removeCase`, `pruneCase`, `checkoutCase` and `reconcileCase` add `hubWorkspacePermittedRoot()` to their `PermittedRoots` (a case with no list gains one); `reconcileCase` takes it because a matrix state that plants at a pair's path can change the listing Reconcile's regeneration writes.
`hubWorkspacePermittedRoot` returns the hub-relative file name from `HubWorkspacePath` on a synthetic Location with an empty `HubPath` and `RepoName` `warp-bare`, this package's fixed fixture repo name, the same synthetic-Location idiom `slugPermittedRoots` uses.
The matrix's mutation oracle keeps holding because every write the regeneration makes (the file, the lock file, `_board/.lyx`) is recorded as Card 1 prescribes.
`checkTaskSlugOpensBareFolder` no longer expects the workspace file to be absent after `AddPair`: it reads the file after `AddPair` and asserts the task spawn leaves its bytes unchanged.
`checkPrimeWorkspaceSurvivesTopologyVerbs` keeps passing unchanged: the file still exists after `Remove` and a `Prune` dry run.

`TestHubWorkspace_TracksTopologyVerbs` (integration, package `fabricengine_test`, one `CopyHub` at anchor `.`, serial: it swaps `addFailAfterWarpHook` and the logger output) reads the file's folder names and resolved paths after each step:
- the copied hub, built through `CloneHub`, holds the file listing the prime's branch and `Bolt`;
- `Add` of `a-task` lists it, its path the pair's anchor dir relative to the hub (the order among pairs is the untagged row's);
- `Add` of `f-task` run from `a-task`'s location (a pair forked from a pair) lists it;
- `AddLocal` of `c-local` lists it;
- `Checkout` of the prime onto a branch the test creates renames the first folder to that branch, and a second `Checkout` back restores the name;
- the file hand-edited, then a `Checkout` of the prime onto a branch whose records sibling branch the test checked out in a second weft worktree (so the weft switch fails and `rollbackSwitch` runs) returns its error, and the file is rewritten to list the prime under its original branch;
- the file hand-edited, then a `Checkout` refused by an uncommitted tracked change in the prime's weft keeps the hand edit;
- `Remove` of `a-task` drops it;
- the file hand-edited, then a `Remove` of a slug with no pair (refused with `ErrPairNotFound`) keeps the hand edit;
- the file hand-edited, then `Prune` without apply keeps the hand edit;
- `f-task`'s warp worktree deleted by hand, then `Prune` with apply drops it;
- the file hand-edited, then an `Add` failing through `SetAddFailAfterWarpHookForTest` returns its error, and the file is rewritten whole without that slug;
- with `FABRIC_SKIP_PUSH` set through `t.Setenv` (as `remove_junctions_integration_test.go` does, so the failed `Remove` respawns no real push), a `Remove` whose weft sibling the test locked with `git worktree lock` returns `weft teardown failed` after the task worktree is gone, and the file does not list that slug;
- the file hand-edited, then an `Add` refused at pre-flight by an uncommitted tracked change in the prime keeps the hand edit (a pre-flight refusal regenerates nothing);
- the file deleted, then `Reconcile` writes it again;
- a directory planted at the prime's `.vscode/settings.json`, then an `Add` succeeds, the file's bytes are unchanged, and the captured log holds a Warn naming `read prime settings` and `lyx fabric reconcile`.

**ImpactSummary:** six fabric verbs gain one best-effort deferred call each, with no change to their results or refusals; the live-state matrix and two ideengine checks absorb the file's new churn.

**Commit:** 2: fabric verbs regenerate the hub workspace file
