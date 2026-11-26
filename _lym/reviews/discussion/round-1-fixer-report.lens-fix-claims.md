```yaml
lens: fix-claims
outcome: reported
```

Lens: fix-claims. I checked each bound, behaviour, count and code fact the fix added or rewrote against the code at HEAD.

1. **MEDIUM** — D4, the added sentence "A failed inverse action is logged, the rest still run, and the original error is returned, as today."
   Summary: today's rollback logs only a failed or refused warp-branch delete; every other failed step is dropped without a log.
   Evidence: `internal/fabricengine/add.go:806-843` (weft worktree removal, junction removal, warp worktree removal) and `:873-877` (`worktree prune`) only set `firstErr`. Only `:860` and `:866` call `logger.Warn`. The callers at `add.go:541` and `:556` discard the return (`_ = t.rollbackAdd(...)`). So "as today" holds for "the rest still run" and "the original error is returned", not for "is logged". The sentence should say that today a failed step is dropped and only a failed warp-branch delete is logged, or else make logging every failure an explicit change.

2. **LOW** — D12, the added sentence "`internal/testkit/boardkit`, which imports `internal/boardengine`, is outside that set."
   Summary: `internal/testkit/envkit` also imports `internal/boardengine`, so it is outside the admitted kit set too, but the record names only boardkit. A reader may take envkit as admitted, especially since D7 lists it beside hubforge.
   Evidence: `go list` gives envkit's imports as `internal/boardengine` and `internal/fabricengine`. No `internal/fabricengine` test imports envkit (count 0), so nothing breaks; the gap is only in the wording.

3. **LOW** — D2, the added sentence "`git.Open` ... refuses every extension but `noop` and `noop-v1` with `ErrUnknownExtension`."
   Summary: when `core.repositoryformatversion` is 0 or unset, `noop-v1` is refused too, and with `ErrUnsupportedExtensionRepositoryFormatVersion`, not `ErrUnknownExtension`. The conclusion for reftable still holds: either error fails the open.
   Evidence: in go-git v5.19.1, `repository_extensions.go`'s `verifyExtensions` checks `extensionsValidForV0` first under version 0. That set lacks `noop-v1` and `refstorage`, and that check returns `ErrUnsupportedExtensionRepositoryFormatVersion`. Only after it does `builtinExtensions` return `ErrUnknownExtension`.

The other claims I checked hold:
- go-git at v5.19.1, with `Open` calling `verifyExtensions`.
- `cmd/lym/main.go` installs both hooks.
- the `LivePair` and `OwnerOf` nil meanings.
- the `upstreamrun.go` done check and the `shedengine.Takeover` fields.
- the code-tip settle re-encoding the status through `state.WriteJSON`.
- the archive tag push's fixed delays (`archive.go:36-37`).
- the lease on the tip the archive read.
- the no-inverse comments.
- no other direct import of `internal/fabricengine` reaches a cut module.
- `internal/fabriccli` hosting both children's entry points.

Tool calls I made:
- `go list` of envkit's and boardkit's imports, a count of fabricengine tests that import envkit, a grep of rollback logging and a read of `archiveTagPushDelays`: my context had no envkit import list, no rollback body and no archive delays.
- a grep to locate `rollbackAdd`: the first grep looked for a plain function, but it is a method.
- a read of `add.go:797-881`: I needed `rollbackAdd`'s error handling to check the "logged" claim.
