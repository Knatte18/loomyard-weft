```yaml
lens: fix-claims
outcome: reported
```

no findings

Lens: fix-claims. I checked that every claim the fix added holds against the code at HEAD.

The fix's added claims hold:
- `lymcwd.NamedWorktree` exists (`internal/lymcwd/lymcwd.go:112`). `lym fabric list` lists the hub's warp worktrees (`internal/fabriccli/fabric.go:170`).
- `Bolt.Pull` (`bolt.go:251`), `retryBoltPush` (`pushclassified.go:678`) and `reconcileRecordsBranch` through `tryRecordsLocks` (`originreconcilemoves.go:62`) hold the Bolt write lock across their moves.
- Fabric pushes Bolt with the rebase-free `gitrepo.PushRebaseFree`, per `doc.go:919-921`. So the stale `bolt.go:262` comment about a coalesced push rebasing the board names no live path.
- "Fabric fires D6's events with no Bolt write lock held" matches today's firing sites:
  - `Fabric.Pull` takes only weft locks (`pull.go:488`, `pull.go:750`).
  - `ReconcileOrigin`'s code-branch moves run under the prime lock (`originreconcilemoves.go:336-392`).
  - `clone.go` takes no lock.
- `OriginReport` is `MutationRecord`, `Unreachable`, `Branches` and `Entries`, with no warning field. Grepping fabricengine's non-test Go files for a warning mutation kind returned nothing.
- `internal/lock` is gofrs/flock: each acquire calls `flock.New` on a new handle, so a second acquire in the same process blocks.
- The `.lym` path is a symlink into the weft worktree, so the support log's filesystem-crossing claim is grounded.
- The docs sections cited ("Instructions seem lost after `/compact`", "Import additional files", "When Claude Code reads AGENTS.md") contain the quoted behaviour.
- `enforcement_test.go` spells `_board` in its token list, its owner-row key and its fixtures. `PATTERN-leaf-packages`'s background for `runproc` lists only the standard library, `proc`, `state` and `logger`.

Calls made:
- `bolt.go:251-262`, `pull.go` lock lines, `MutationRecord`, `CloneHub`: to check the lock span and the "no warning" claim, which my context lacked.
- The coalesce/rebase grep and `pushclassified.go:670-700`: to test whether any Bolt move escapes the write lock.
- `doc.go:908-922` and the `PushCoalesced` callers: to confirm Bolt pushes are rebase-free.
- `spawn.go:236-248`, `clone.go` and the pull/topology lock grep: to confirm the seam's firing sites hold no Bolt write lock.
