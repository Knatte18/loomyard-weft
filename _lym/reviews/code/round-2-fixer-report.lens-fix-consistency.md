```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F2 change set for consistency with its surroundings.

No findings that reach MEDIUM or BLOCKING. One LOW:

- **LOW**, `internal/shedarbiter/edits.go:41`. The `readEdits` doc line reads "An uncommitted change is a tracked file or an untracked one whose path or content differs from the spawn's." That wording is loose. Tracked paths are compared through `Dirty`, and every path through `Changes`. It does not say that both maps are compared. It is no longer wrong, because it now names content as well as path, but "a tracked file" with no qualifier overstates what the first comparison covers.

I checked these and found them consistent:

- **Seam type**: the `Changes` type is `map[string]string` in `WorktreeSeam`, `ArbiterAttempt`, the `fakeWorktree` fixture and the `loomcli` seam. The `samePaths` helper and the `slices` import are gone from `edits.go`. The only other `.Changes` callers are in unrelated packages.
- **Docs**: the `WorktreeSeam.Changes` and `ArbiterAttempt.Changes` docs both say the map holds content hashes and "deleted".
- **Path keys**: the keys are relative to the code worktree, as the docs say. The `loomcli` seam joins them onto `location.WorktreePath()` before hashing. `worktreeSnapshot` uses the same root for its prefix check. Both seam reads now go through the shared `hashFileOrDeleted` helper.
- **Persisted attempts**: the old `[]string` form of `changes` in `attempt.json` cannot break reading. `main` has no `Changes` field, so no released record holds the old shape.
- **Messages**: the new Changes-read refusal message is distinct from the `Snapshot` refusal. The existing row's `wantRefusal` substring still matches only the `Snapshot` message, so the two rows can tell the reads apart.
- **Test**: the fixture passes `spawnChangesErr` as `changesErr`, which `onWait` only overwrites after the refusal.
- **Missing sibling**: nothing mentions the old path-list wording in the contracts or `docs`. The summary's "What changed" bullet that F2 asked to update sits under `_lym`, which this round does not touch, so I did not check it.

Tool calls:
1. A `grep` over `internal`, `contracts` and `docs` for path-set wording and for other `Changes:` seam users, because my context did not show twins outside the files read.
2. A `git grep` of `main` for `Changes` in `internal/loomengine/arbiter.go`, to see whether the `[]string` shape had ever been released.
