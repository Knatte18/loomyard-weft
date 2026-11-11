# Remove the hub's _launchers directory: nothing in lym runs its scripts

`remove-launchers-dir` · Task · enhancement, fabric, cleanup

Depends on: [`worktree-create-build-leftovers`](design-worktree-create-build-leftovers.md)

Imported from [issue #95](https://github.com/Knatte18/loomyard/issues/95).

Seen (orch, 2026-11-10): internal/fabricengine writes <hub>/_launchers/<anchor>/<slug>/{ide.sh,run.sh,fabric-checkout.sh} for each pair, plus <hub>/_launchers/ide-menu.sh, and removes them with the pair (addanchor.go, reconcile.go, anchorchange.go, destroy.go). A read of the non-test code finds no caller that runs, reads or documents those scripts; only the writer and the cleanup of them use the directory. The operator says terminals open through reed and the directory is no longer used. Leftovers of pairs that no longer exist stay behind, for example the pairs cold-gate-fixer and fabricengine-tier1-timeout-load of 2026-11-02. Idea: take the launcher scripts, ide-menu, the portal launcher test and the reserved _launchers name out of fabricengine in one rename-free removal, and make lym cleanup remove an existing _launchers directory once. Check first whether any desktop shortcut or VS Code task of an installed hub still points at one of the scripts.