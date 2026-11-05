```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix against each finding whose premise is a set of callers or sites.

**Sites checked, none to report:**
- **F5/F7, `PairTaskBranch` callers** (`PairTaskBranch(` in non-test code): `internal/battencli/wire.go:1032` still uses the local-only read. A worktree directory is present there, so it never runs after the local parts are gone, and the local read is correct. `remove.go:540` is `inspectPair`, and `taskbranchdiscard.go:49` already falls back to `originTaskBranch`, so both are covered. `pullrequest.go:45` and `readopt.go:55` now use `PairTaskBranchOrOrigin`.
- **F5/F7, `wire.go:1065`:** `createRefusal(err, slug, cfg.BranchPrefix+slug, ...)` builds the name of a new pair's branch after `Add` failed, so the current prefix is right there.
- **F4:** `refuseWhenNothingRemains` has one caller, `refuseRemoval`. `removePair` now takes the origin answer from the refusal. `localPartRemains` is the only place that lists the local parts, and no inline copy is left.
- **F9:** no `ownedManagedBranch(l, ` comment or call is left anywhere in `internal`.
- **F3, other uses of `BranchPrefix` in non-test code:** the remaining reads are `pairBranchNames` and `slugOfBranch` in `pairbranch.go`, plus `add.go:194` for a new pair's branch, which should use the current prefix. No other site re-derives a live pair's branch from the current prefix.
- **F3, branch-to-slug call sites:** the sites that use `slugOfBranch` are `TaskBranchSlugs`, `keptReason`, `deleteLeftoverUnderLock` and `cleanupOrphan`. Your fix has rows for `TaskBranchSlugs`, `keptReason` (through the sweep) and `cleanupOrphan`.

**Findings:**

1. LOW — `deleteLeftoverUnderLock` (`internal/fabricengine/cleanupremotewarp.go:147`) is covered only partly by the F3 test. Its `slugOfBranch` call decides which slug's teardown lock is taken. `TestCleanupRemoteWarp_BranchPrefixSetOverBareBranches` applies deletes with a prefix set, but the lock slug equals the branch in the bare case, so a wrong slug derivation would not fail the test. The review named only `keptReason` here, so this is minor.

**Not checked:**
- F1, F2, F6, F8 and F10 are not set findings.
- I did not check whether other docs outside `template.yaml` and `doc.go` still say the prefix forms every pair's branch, because `doc.go` was already updated and a wider doc search was beyond my 6 calls.

**Calls made:**
1. Bash grep for `PairTaskBranch` callers, non-test `BranchPrefix` uses, stale `ownedManagedBranch(l, ` and inline local-remnant tests. My context lacked a repo-wide search for these.
2. Read `internal/battencli/wire.go:1018-1070`, to judge the `PairTaskBranch` caller and the `BranchPrefix+slug` site that the grep surfaced.
