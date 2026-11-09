```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the review's set searches (callers and sites under one key) against the fixed tree and check the fix covers every member.

No findings.

- **F1 (`runproc.EndProcess` sites):** the only production caller left is `internal/tendercli/deps.go:117`, the `lym tender reap` path `PATTERN-child-processes` allows. No stale "ends it again" or "ends the process again" wording remains in code or docs.
- **F3 (stop and end sites):** `endDobbyRoles` has no remaining reference. The other ends of a `RoleDobby` record (`internal/loomcli/pawlwiring.go:429`, `internal/pairteardown/skip.go:94`) are beam teardowns. The prime's refresh is not owned by a beam slug, so they need no change.
- **F3/F6 (docs naming the refresh):** `internal/orchcli/doc.go`, `internal/beamcli/doc.go`, `internal/dobbyengine/doc.go` and `internal/fabriccli/doc.go` agree with the new behavior. `docs/overview.md:807` describes only the start and the single-flight condition. The fix leaves it true.
- **F6 (anchor move):** `EndDobbies` is the only site that ends the prime's dobbies on a move, and `dobbyPassInFlight` is the only wait. Both are covered by the new `fabriccli/doc.go` text.
- **Unchecked:** F5, F7, F8 and F9 are test and count sets confined to the files the fixer edited. I did not search them separately.

Tool calls:
1. One `grep` batch over the tree for `EndProcess`, `endDobbyRoles` and "ends it again". It also covered the `test-map` and `RoleTestMap` docs and the `RoleDobby` end sites. My context lacked repo-wide search results for these sets.
