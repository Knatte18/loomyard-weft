```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking that the fix covers every member of each set-premised finding.

Unchecked: F8 (the set of formats other binaries read) was not re-searched. F3, F4 and F6 have single-site premises, so there was no set to rerun.

**1. BLOCKING: F5 change, `reach.go` `caveats` → `unmappedTierNames(tests, mappedTierConfigs(s.cfg))`**

`lym gate reach <file>#<Func>` now reports tier0 and tier1 as tiers no map holds.

- `describe` has two callers. The second is `runReachFunction` (`internal/gatecli/reachverb.go:188`).
- The step it receives is built at `reachverb.go` just above the `testreach.Open` call as `reachStep{location: location, root: location.WorktreePath()}`, with no `cfg`, although `cfg` is loaded in that function.
- With a zero `gateslot.Config`, `mappedTierConfigs` returns an empty map. Every tiered test then counts as unmapped.
- So for any function no test reaches, the verb prints "tests of p in tier0 and tier1, which no map holds, may run it unseen". This is false, and it is the kind of misleading cause the task set out to remove.
- Before the fix, `unmappedTierNames` used the package-level `mappedTiers`, so the verb was right.
- The gate stayed green, so no test asserts this caveat on the verb path.
- Fix direction: set `cfg: cfg` on the verb's `reachStep`, and assert the verb's exact caveats in `reachverb_integration_test.go`.

**2. LOW: F1 change, the format check sits only in `staleLinked`**

An older-format worktree map that is fresh by hashes still wins over a newer shared map wherever no refresh replaces it.

- `testreach.worktreeMapWins` (`reader.go:96-101`) returns true on `Fresh` alone, whatever `Format.Version` says.
- `lym gate reach` never refreshes. A step whose refresh was refused (dirty file, budget, no slot) also reads that map.
- In both cases the old map's empty unresolved entries still hide the call, even when the shared map has been rebuilt at version 3.
- The summary's "The reach refresh rebuilds an older worktree map only for a package holding a changed function" does not cover the verb or a refused refresh.
- The other freshness reader, `freshMaps` (`mutate.go:344`), also ignores the version. That is harmless, because mutation uses only `Measured` entries.

No other members found for F7 or the F1 refresh prose: the search turned up no further stale "records no coverage" or "absent or stale" wording about these maps.

Tool calls:
1. grep for `Fresh`, `freshMaps` and `ReadPackageMap`: my context lacked the full list of freshness readers.
2. Read `reachverb.go:150-200` and grepped the `describe`/`caveats` callers: my context held only grep lines for the verb.
3. grep for stale coverage and refresh prose across `internal` and `pattern`: this reran the F7 and F1 searches, which my context had not done.
