```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set-premise searches for F1 and F6 and check the fix covers every member.

**Findings:** no findings.

- **F1** (sites naming the destruction-chokepoint exception): the only files that state it are `pattern/PATTERN-fabric-git.md` and `internal/fabricengine/doc.go`. Both were amended. No contract, stencil or other PATTERN entry restates it. Every caller of `removeWorktreeLeftover` is in `destroy.go` or `remove.go`, and `remove.go:337` and `remove.go:960` follow a gated git run, so the narrow doc comment holds for them. `RemoveStrayTaskPath` at `destroy.go:1441` is the only caller without one, and the amendment names it.
- **F6** (sites for the cancel envelope key and `StrayPathDeleted`):
  - `stray_directory_deleted` appears in the `cancelcmd.go` help text and in the `cancelEnvelope` map, in the same position. The other cancel tests read `pair_removed` and other keys by name, so none enumerates the full key set.
  - `RemovePair` is reached through `pairteardown.Run` (`teardown.go:522`, result passed through whole), `wire.go:1046` and `cancel.go:669`. `ClearStray` is set at only those two request sites.
  - Both `wire.go` and `cancel.go` read the result, so `StrayPathDeleted` is covered everywhere it can be set.

**Unchecked:** F2, F3, F4 and F5 have no set premise. Each is a single-site coverage gap, so I ran no search for them.

**Tool calls (2):**
1. A `grep` across `.md`, `.go` and `.yaml` for `removeWorktreeLeftover` and "one exception", plus a repo-wide `grep` for the cancel envelope keys. My context held only the files the fixer read, not the rest of the repo.
2. A `grep` for the destroy-chokepoint symbol files, `RemovePair` callers and `ClearStray` setters. My context did not show every caller of those symbols.
