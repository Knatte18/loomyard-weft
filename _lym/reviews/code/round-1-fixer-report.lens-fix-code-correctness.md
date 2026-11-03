```yaml
lens: fix-code-correctness
outcome: reported
```

no findings

Traces I ran:
- **`presentTaskWorktreePath`, stray directory:** `taskWorktreePresent` stats the path, which succeeds, and `ResolveWorktree` fails, so it returns `(false, nil)`. The function then returns `absentTaskWorktreeRefusal`, the by-name refusal. That was F1's goal.
- **`presentTaskWorktreePath`, missing path:** `taskWorktreePresent` returns `(false, nil)` and the same refusal comes back.
- **`presentTaskWorktreePath`, stat error other than not-exist:** the error comes back unchanged.
- **Resolved path:** `taskWorktreeLocation` resolves it a second time. That is redundant but harmless.
- **`taskWorktreeLocator`, first call:** it goes through `taskWorktreeLocation`, so it now also refuses a stray directory.
- **`taskWorktreeLocator`, cached branch:**
  - It stats only, so it spawns no git.
  - A missing path gets the same refusal.
  - Another stat error is returned.
  - Hiding `.git` after the first resolution keeps returning the cached location, as `stepTaskWorktreeSeams_RunGitOnlyUntilTheFirstResolution` expects.
  - Before the first resolution, `.git` hidden gives a refusal, which satisfies that test's line 1724 check (`err == nil` is fatal).
- **New subtest `MarkedPairOfATaskWhoseWorkReachedItsParentPasses`:**
  - With the real wiring, `ReachedParent` gets `found=true` and `CurrentProducer` set to `NameWorktreeTeardown`, so `childReachedParent` returns true whatever the child's status reads.
  - `promoted` becomes false and `refuse` passes.
  - If `ReachedParent: hubChildReachedParent(...)` is dropped, the seam is nil, `promoted` stays true, and `refuse` returns the beam-mark refusal, so the subtest fails.
  - The sibling subtests run sequentially on the shared hub, with no shared mutation.
- **Gate:** tiers 0 and 1 for `./internal/battencli` were green after the second edit. No other caller of `presentTaskWorktreePath` exists; a grep shows only `taskWorktreeLocation` and the locator.

Tool calls beyond the context: none.
