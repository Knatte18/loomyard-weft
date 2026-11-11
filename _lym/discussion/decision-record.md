## Goal

```markdown
Imported from [issue #95](https://github.com/Knatte18/loomyard/issues/95).

Seen (orch, 2026-11-10): internal/fabricengine writes <hub>/_launchers/<anchor>/<slug>/{ide.sh,run.sh,fabric-checkout.sh} for each pair, plus <hub>/_launchers/ide-menu.sh, and removes them with the pair (addanchor.go, reconcile.go, anchorchange.go, destroy.go). A read of the non-test code finds no caller that runs, reads or documents those scripts; only the writer and the cleanup of them use the directory. The operator says terminals open through reed and the directory is no longer used. Leftovers of pairs that no longer exist stay behind, for example the pairs cold-gate-fixer and fabricengine-tier1-timeout-load of 2026-11-02. Idea: take the launcher scripts, ide-menu, the portal launcher test and the reserved _launchers name out of fabricengine in one rename-free removal, and make lym cleanup remove an existing _launchers directory once. Check first whether any desktop shortcut or VS Code task of an installed hub still points at one of the scripts.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-11): TestSmokeBootstrapLifecycle died_driver failure is main's own, left unfixed here

Decision: internal/loomcli TestSmokeBootstrapLifecycle/died_driver_proceeds_to_the_success_envelope_and_logs_why fails on main since d1535412, because the driver log now says 'step interrupted (loop-exited)' where the test waits for 'invalid state'. This task leaves it unfixed and touches no shed, loom or driver code.

Rationale: The failure is outside the task's scope. It is filed as the bug note smoke-died-driver-wording on the board.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
