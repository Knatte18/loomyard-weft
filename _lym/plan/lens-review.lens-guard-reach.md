```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal and test row the plan adds can be reached from a real entry point.

**1. BLOCKING — Card 9 (D5 "every verb that loads loom/tapestry config runs `EnsureOrWarn` first").** Card 9 adds the call only to `internal/loomcli#loomCLI.arm`. The `lym loom circling` group never passes through `arm`. The parent pre-run skips the circling, decision, parent-review and peers groups, and also checklist and pusher (`internal/loomcli/cli.go:261-268`). Circling resolves its own worktree in `circlingState.preRun` (`internal/loomcli/circling.go:179`). `perchHardCap` still runs strict `loomengine.LoadConfig` and `LoadTapestryConfig` (`internal/loomcli/circling.go:262`, `:267`). So the guarded load recurs on a path the card does not guard.
- Effect on the test: Card 9's own test, `TestArm_StaleStampReconcilesBeforeANonStartVerbLoads`, drives "`lym loom circling` reading the code perch's caps". On that path no walk runs, so card 2's strict load refuses `code_max_bounces: 2` and the test cannot pass.
- Fix: add the `EnsureOrWarn` call to `circlingState.preRun`, or to a helper every self-resolving group's pre-run calls, and list it on card 9. Or move the test onto a verb that `arm` actually covers.

**2. LOW — Card 8 (D7 "a walk whose board write fails leaves the entry as it was").** The `beforeRunSettingsCarry` hook fails before `RewriteRunSettings` runs. `EnsureFailingRunSettingsCarry` therefore reaches only the skip-and-stale-stamp branch. Nothing reaches the case where `RewriteRunSettings` has rewritten entries and then its save, commit or push fails. That leaves the "board as it was" restore of a real failed write untested.

Everything else checked is reachable:
- Card 2: local, default and worktree refusals, the `Set` refusal and the `Unset` admission. `lym config reconcile --apply` goes through `ReconcileAll`, whose `TemplateUnder` reads only the base file, so the move runs before any refusing read. `--unset` with a failing "before" load is let through by `newLoadFailure`.
- Card 5: the run_settings, profile and setting-key refusals.
- Card 6: worktree, local, hub-base and prime-seed moves.
- Card 7: the seed-only profiles branch.
- Card 8: the carry after a committed hub-wide write, and the skip on a skipped write.

Tool calls beyond my context:
- Read `TemplateUnder`, `baseDocument`, and configcli's `setChecked`, `newLoadFailure` and `runReconcile`: to check whether a reconcile or `--unset` reads the refusing local layer before the move.
- Grepped loomcli for its commands and their arm/pre-run wiring, plus the loaders in circling, validate and prfindings: to find loom verbs that bypass `arm`.
- Read `cli.go`'s `resolvePersistentPreRun` and `circlingState.preRun`: to confirm which groups skip `arm`.
- Grepped the skipped groups and all packages for loom/tapestry loaders: to find other unguarded load sites.
