```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked that each claim the fix added or rewrote holds against the code at HEAD.

no findings

The claims I checked, and what held each one:
- **D9's `<perch>_auto_accept` is Guard:** `internal/reviewbudget/reviewbudget.go` `Categories` sets `AutoAcceptKey(perch)` to `CategoryGuard`.
- **D9's loom fixer and split fans are Guard:** `internal/loomengine/config.go` sets `discussion_fixer_fan`, `code_fixer_fan`, `split_reviewer_fan` and `split_fixer_fan` to `guard`.
- **D9's `tree_scope` is Guard and set by no run_settings:** that is what the `internal/darnengine/config.go` `ConfigSpec` doc says.
- **D9's local layer holds only CategoryNone keys:** `internal/configengine/doc.go`, section "The local rule".
- **D9's seam refuses a run_settings Guard key with a hub-file way forward through `HubOnlyWayForward`:** `internal/configengine/layers.go:88-110` declares the map. `internal/configreg/runsettings.go:43` and `:114-118` (`refuseHubOnlyKeys`) refuse such a key with that text as its way forward.
- **D6's auto-accept turns a spent budget's CONTINUE or CIRCLING round into an accept:** the `internal/shedadapters/bouncer.go` `NewBouncer` doc says so.
- **D4's `acquireSlot` takes a tier's slots through `gateslot.Pool.AcquireSlots`:** `internal/gatecli/run.go`, `acquireSlot`.
- **D4's acquire is skipped only for an inherited slot:** `exe.inherited` is set only when `os.Getenv(gateslot.InheritEnv)` names a held slot, and `InheritEnv` is `"LYM_GATE_SLOT"` (`internal/gateslot/pool.go:27`).
- **D4's CPU watchdog and memory guard exist per call:** `run.go`, `watchdogApplies` and `memoryGuardApplies`.
- **D2's Preflight reads tiers the way `lym gate test` narrows `{test_targets}`:** `pattern/PATTERN-test-speed.md:36-38`, under Four tiers, and `internal/gatecli/doc.go:105`.
- **D5's `lym fabric remove` ends a pair through `internal/pairteardown`:** `internal/fabriccli/remove.go:87-96` calls `r.teardown(ctx, pairteardown.Request{...})`. The dobby orphan and handed-over removals go through `pairteardown.RemoveOrphanPair` and `RemoveHandedOverPair` (`internal/beamcli/dobbydeps.go:443-446`), so a hook in pairteardown reaches those routes too.

Tool calls beyond my context:
1. I grepped `internal/configreg/runsettings.go`, `internal/configengine/layers.go` and the callers of `pairteardown` and Worktree-Teardown. My context did not show how `HubOnlyWayForward` reaches the run-settings refusal, or which routes call pairteardown.
2. I grepped the exported functions of `internal/pairteardown`, the pairteardown calls in `internal/fabriccli/remove.go` and the loomcli uses. My context did not show which removal routes enter pairteardown.
