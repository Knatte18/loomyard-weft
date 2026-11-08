```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD (fix-claims lens).

1. **LOW**, D7 step 1: "so a renamed or moved file's unchanged tests are not taken". The claim is broader than the mechanism. The lookup goes by package and name across "the package's `_test.go` files at the merge base", so a test whose file moved to another package directory is not found there and is taken as new. Only a rename or move within the package leaves tests untaken, and the new D7 criterion says exactly that: "A test file renamed or moved within its package leaves its unchanged tests untaken". The step's sentence and its criterion disagree on moves across packages.

2. **LOW**, D7 step 1: "the step reads the package's `_test.go` files at the merge base through `fabricengine.CodeFileAt` (new), which reads a file of the warp checkout at a revision". A rename lookup has to know which `_test.go` files the package directory held at the merge base. A single-file read at a revision cannot enumerate them, and the record names no route that lists a directory at a revision. Before the fix, step 1 read only paths that `impactset.ChangedPaths` named, so it never needed a listing. `ChangedPaths` (`internal/impactset/targets.go:102`) does not give that set either, since it lists only changed paths.

Checked and holding:
- D6 / F2: `EndRole`'s contract ends the processes below the records it ends (`internal/runproc/end.go:71`).
- D6 / F2: the prime dobby's stop is `orchcli`'s `endDobby` with `owner = runproc.HubOwner` when not a beam (`internal/orchcli/cli.go:271`). The other `RoleDobby` stops, `loomcli/pawlwiring.go:429` and `pairteardown/skip.go:94`, are beam-scoped by slug. The anchor route `EndDobbies` uses `EndSparing` sparing only `RolePush` and `RoleSync` (`internal/fabriccli/anchor.go:468`), so it would end a `test-map` record as well. Nothing contradicts the new sentence.
- D6 / F2: `runproc.RoleTestMap` does not exist in `internal/runproc/record.go`, which is consistent with D6 marking it "(new)".
- D1 / F9: `cmd/testtiming/spawnscan.go` exists, so the move claim names a real source.
- D5 / F4: the "cannot stall" claim rests on the fixed grace bounding the wait, not on the group kill. It holds even for a descendant that left the group.
- D5 / F4: the per-package lock is a skip-if-held lock (D5), so the restated time bound needs no lock-wait term.

No other added or rewritten sentence asserts a fact about the code at HEAD.

Tool calls:
- `grep` for `RoleDobby` users and the `runproc` role constants: to find every route that stops the prime dobby and to confirm `RoleTestMap` is absent. My context held only `orchcli`'s route.
- `sed` of `internal/fabriccli/anchordeps.go` around `DobbyPID` and `EndDobbies`, plus a hibernate grep: to see whether another stop route spares or ends a `test-map` record. Not in my context.
- `grep` for `EndDobbies(` callers and `git diff --stat`: to find the spare list passed on the anchor route. Not in my context.
