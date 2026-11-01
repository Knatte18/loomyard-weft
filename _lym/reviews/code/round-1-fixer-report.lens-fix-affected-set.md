```yaml
lens: fix-affected-set
outcome: reported
```

Rerun of each set-premise finding's search; the fixes cover the named members, with two siblings left stale.

**LOW — stale sibling doc (F5)**
- Changed text: `internal/fabricengine/archive.go:160`, `resolveArchiveTip`, which no longer reads `FETCH_HEAD`.
- Summary: `internal/fabricengine/doc.go:1214` still says "`resolveArchiveTip` resolves `FETCH_HEAD^{commit}`, revision syntax over git's fetch output". That is now false. The function fetches the `ls-remote` SHA by SHA and runs no `rev-parse`.
- Evidence: the FETCH_HEAD grep shows no `rev-parse FETCH_HEAD` in `resolveArchiveTip` any more. The bullet sits in a list of `rev-parse` spawns that survive, so it now lists a spawn that is gone.

**LOW — stale sibling doc (F4)**
- Changed text: `PushRebaseFree`'s doc, which no longer says "Lock-free".
- Summary: `internal/fabricengine/doc.go:299` still calls it "the lock-free, rebase-free `PushRebaseFree`". The same wording survives in the test message at `internal/fabricengine/pushrebasefree_integration_test.go:57` ("lock-free push"). Both refer to the single-pusher lock, so they are not false, but they now read like the sentence F4 corrected. `internal/fabricengine/pushanchored.go:69` says "lock-free status push" and is likewise ambiguous.
- Evidence: the `Lock-free` grep.

**Sets checked, no finding**
- **F1/F9, push sites:** the only push outside `internal/gitrepo` is the tag push at `internal/fabricengine/archive.go:128`, which the narrowed `PATTERN.md` entry now names as exempt.
- **F1/F7, fetch sites:** no `"fetch"` literal exists outside `internal/gitrepo` and `internal/fabricengine`, apart from the `transport.go` and `cluster.go` verb tables. The new guard's `internal/fabricengine` scope therefore covers every member.
- **F7:** the two allowlisted files (`recordsprobe.go`, `originreconcile.go`) and the `--refmap=` sites in `remoteleftover.go` match the grep.
- **`remoteleftover.go` FETCH_HEAD reads:** these follow an unlocked `--refmap=` fetch. That is exempt by the entry and not the F5 race, since they write no tracking ref.

**Not checked:** F2's method set. I did not recount the `withRemoteRefs` methods against the table rows. `withRemoteRefs` appears in `pull.go` (`Pull`, `Fetch`), `push.go` (the push methods) and `remoterefslock.go` (`FetchWithArgs`); the table covers all of them except `PushCoalesced`, which reaches the lock through `pushWithRebaseRetry`.

**Further calls**
1. Read `internal/fabricengine/doc.go:1205-1219` to see whether the `resolveArchiveTip` bullet's context still holds.
2. Ran a combined grep for `"Lock-free"` and "lock-free" across `internal/gitrepo`, `internal/fabricengine` and `PATTERN.md`, and counted files; the context held no such search.
