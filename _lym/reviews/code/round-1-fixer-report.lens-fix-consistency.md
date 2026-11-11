```yaml
lens: fix-consistency
outcome: reported
```

No findings.

- **`outcomeKept`:** it appears in no doc, spec or `recoveries` text, and it has no twin outside `set.go`. The `outcomeNothing` and `outcomePushed` users are `setName`, whose upgrade branch already tests only `outcomeNothing` (`set.go:264`), and the existing tests. The `sweepHub` doc comment now matches the new behaviour.
- **`runSet`:** it counts a hub as unfinished through `len(hub.Lost) &gt; 0`, so the exit stays non-zero for a kept hub. The new test asserts that.
- **`OwnerRequest` comment:** the new sentence agrees with the `Replace` field comment and with the `SetOwners` doc comment.
- **Replace-guard test:** its rows match the guard condition at `ownerset.go:97`. The `WithClaim` row passes the exactly-one-slug check first, so it reaches the replace guard.

Tool calls:
1. `grep` for `nothing to sweep`, `outcomeNothing` and `outcomePushed` across the repository, excluding `.lym`, `_lym` and `.git`, to find docs or twins outside the changed files that my context lacked.
