---
round: 1
focus:
  - "internal/fabricengine/failedpush.go `lymVerbDir`: settle whether it returns the right directory for every caller, including a records worktree path, a prime, and a worktree whose marker is absent or unreadable, and whether `lymcwd.NamedWorktree(filepath.Dir(root), filepath.Base(root))` finds the marker the way the cwd gate does."
  - "internal/fabricengine/originreconcile.go `stopWayForward` `BranchRewritten`: the code side now names the anchored directory while the records side still names `b.Worktree`; settle whether the records branch is correct as left or a `lym` verb that the cwd gate refuses."
  - "Other `lym <verb>` way-forward lines that still name a worktree root, such as internal/fabricengine/anchorstop.go (`lym fabric anchor ... in %s`), addparent.go, addadoptonly.go, internal/adoptcli/adopt.go, internal/battencli/readoptstop.go: settle whether each is within the issue's scope (the failed-push notice) and whether the commit message's claim that every lym step now names the anchored directory holds."
  - "internal/fabricengine/failedpush_test.go: settle whether the four new anchored rows add coverage the existing rows lack, and whether `pull.go`'s `recordsConflictWayForward` push step and the `BranchRewritten`, anchor-stop and dirty paths in originreconcile.go have any test for their new `lymVerbDir` call."
  - "internal/fabricengine/doc.go addition: settle whether it matches the diff's behavior and whether docs the repository's constraints name for this kind of change landed in the same diff."
---
