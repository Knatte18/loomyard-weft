```yaml
lens: fix-proposal
outcome: reported
```

**Fork findings**

1. **MEDIUM** — `internal/stencilstore/skew.go`, `refuseNewerBoardCopy` and `writtenByNewerBinary`. The registered ordering runs before the cheap body-hash check, so it costs git work on common reads.
   - The check order is `!writtenByNewerBinary(...) || BodyHash(onDisk) == BodyHash(def)`.
   - `writtenByNewerBinary` calls the registered `Older`, which runs `SHAExists` twice and `IsAncestor` (a `git merge-base` subprocess, `internal/gitrepo/ancestry.go:29`). There is no caching and no `sync.Once` around the successful path.
   - It does so whenever `recorded.Revision` is non-empty and differs from the running revision.
   - A refresh rewrites only the stencils whose body changed, so most board copies keep an older writer's revision. Every `Read` of one, and every plain include and engine block, therefore runs that git work, although the body equals the shipped default and the answer cannot matter.
   - Checking `BodyHash(onDisk) == BodyHash(def)` first would skip the git work for unchanged stencils.
   - F4's proposal puts the ordering in the render path, and this ordering of the two checks makes it expensive.

2. **MEDIUM** — `internal/stencilstore/skew.go`, `NewerBoardCopyError.Error` and F1's refusal rule. The way forward is wrong for a freshly started older binary.
   - Any differing body from a newer writer is refused, including a harmless wording change, and an old binary's seed pass never refreshes it (`RecordedNotOlder` leaves it untouched).
   - Take a stale or branch-built lym that is started fresh against a hub whose board a newer production binary refreshed. It now fails on every stencil that binary changed.
   - Its error says "restart the run's driver on the current lym binary". That advice fits only a long-running driver. For a fresh old binary, a restart changes nothing until the binary itself is upgraded.
   - F4's rule makes it wider: an unrelated-branch writer answers `RecordedNotOlder`, so a feature-branch build is refused on main's copies.
   - The old behaviour ran such a binary on the newer text, and it failed only when the text was incompatible.
   - The error text should name upgrading the binary as well as restarting the driver.

3. **LOW** — `cmd/lym/stencilseed.go`, `seedStencils`. The ordering is registered only after the early returns, so the detached daemon, which opts out of the seed pass, never gets it.
   - Those returns are `testing.Testing()`, `skipStencilSeed(cmd)`, and a missing hub.
   - Its comment says the opt-out matters for a long-lived process such as the daemon, which is the process the skew refusal protects.
   - Such a process falls back to commit-time ordering, which is the F4 gap again, so F4's premise is only partly resolved.
   - The registration belongs where `RegisterShipped` runs or in the root pre-run before the skip, not behind the seed pass's gates.

I checked whether `lym stencil diff`, `promote` or `pins` read through `stencilstore.Read` and would now refuse a newer copy. The grep over non-test sources in `stencilcli`, `validate`, `diff`, `promote` and `pins` returned no `Read` callers, so I found no finding there.

**Tool calls**
- `grep` for `stencilstore.Read*(` callers in stencilcli, validate, diff, promote and pins: my context lacked the callers outside `internal/stencilstore`.
- `grep` of `gitrepo.SHAExists` and `IsAncestor` (same command): my context lacked their cost, which is a git subprocess per `IsAncestor` call.
