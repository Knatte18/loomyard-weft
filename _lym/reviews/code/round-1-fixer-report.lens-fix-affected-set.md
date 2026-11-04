```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F4 and F6 fixes covered every site of their sets, and whether F2 left `holdNothing` callers that should count.

**Findings**

- **LOW**, `internal/battenshed/primelockwait.go:1-2`, the reworded header.
  - **Summary:** The new header names Worktree-Teardown and Worktree-Create's `HoldPrimeLock` as the `acquirePrimeLock` callers, but a third caller exists.
  - **Evidence:** `internal/battenshed/land.go:244` calls `release, lockReason, err := acquirePrimeLock(ctx, p.name, p.slug, p.primeLock)`.
  - **Context:** The old header ("both prime-lock producers") left Land out too, so this predates the fix. The rewrite still names two callers and omits one. Keeping the "name the group" rule from `prose` would mean wording it as "the prime-lock producers" without a list.
  - **Reach:** `internal/battenshed/doc.go:178` ("The two producers that hold the hub's prime lock") has the same gap. The review didn't name it and the fix didn't touch it.

**Sets checked with no finding**

- **F4, "never around a network call" claims:** the grep over `internal/battenshed`, `internal/battencli`, `internal/fabricengine` and `docs` found no remaining claim of that kind about the create hold.
  - The reworded sites are `create.go:36-37` and `doc.go:182-183`.
  - `wire.go:939-940` says "a create waiting on origin", which is accurate.
  - `fabricengine/add.go:64` says "no network wait of the probes", which is also accurate.
  - The other "no network call" hits describe local reads unrelated to the hold.
- **F2, `holdNothing` callers:** the pre-fix grep listed these remaining callers: `wire_seams_test.go:661`, `beamland_integration_test.go:94`, the other `lifecycle_integration_test.go` rows (978, 1000, 1039, 1084, 1111), and the non-held `beamfork_integration_test.go` rows.
  - None of them tests where the hold is taken, so they keep `holdNothing`.
  - The three places the review named all now use `countingHold`.

**Unchecked:** F1, F5 and F7 have no set premise.

**Tool calls**
1. One `grep` combining the "network call" and `acquirePrimeLock` searches across `internal/battenshed`, `internal/battencli`, `internal/fabricengine` and `docs`. My context held no search of either set after the edits.
