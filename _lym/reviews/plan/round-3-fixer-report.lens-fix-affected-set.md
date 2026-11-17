```yaml
lens: fix-affected-set
outcome: reported
```

Lens: fix-affected-set. I re-ran F1's search for every reader, writer and mention of the owner pause seam's members.

List of findings left unchecked: none. F1 is the review's only finding, and I checked it.

No findings.

The fix covers every member of F1's set:
- `ownerPause` and `ownerPauseMu` at `internal/orchcli/ownerpause.go:26-27` are now on the Delete list. Their only readers and writers are the two deleted functions (`ownerpause.go:33-44`).
- The header line at `ownerpause.go:2`, "the owner pause seam", is covered by the new Intent clause.
- `internal/orchcli/doc.go:236` (`InstallOwnerPause` stores `pauseOwned`) is covered by `internal/orchcli/doc.go` on the Edit list and by "Each package doc drops the owner pause". The fix did not touch it.
- The `installedOwnerPause()` call at `internal/orchcli/cli.go:278` and the install at `cmd/lym/main.go:167` were already covered by the card's `pauseOwned` replacement and `newRoot` edit. The fix did not change them.
- The `sync` import goes with `ownerPauseMu`, which is its only user. As an import rather than a symbol, it needs no card target.
- `ownerPause` in `internal/loomcli/ownercheck_test.go:44-124` is an unrelated test-local bool field, so it is outside the set.
- The `docs/research/**` hits are archived bench reports, also outside the set.

Tool calls:
- One `grep` over `*.go` and `*.md` for `ownerPause`, `ownerPauseMu`, `owner pause seam`, `InstallOwnerPause` and `installedOwnerPause`. My context held the fixer's earlier grep, which covered only `*.go` and did not search the header phrase.
