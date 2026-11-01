```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch and test row the cards add, I checked whether a real entry point can reach it, and whether the guarded site also occurs on a path the card leaves unguarded.

Findings:

1. **MEDIUM — Card 12 (D7), the ended-run set-aside branch: no guard against an empty run list.** Card 12 sets a claimed commit aside, or skips its ledger part, "when every one [of its ledger runs] is in the live ended set". A commit that touches only `heddle.json`, or only `README.md`, has no ledger runs, so that test is vacuously true.
   - Today's rule only fires when the slug list is non-empty: `internal/fabricengine/replay.go:769` checks `len(slugs) > 0 && …`.
   - As the card is worded, every heddle-only commit would have its "ledger part skipped and the original tagged". That would ship a wrong set-aside tag on ordinary heddle commits.
   - The card should state that the rule needs at least one ledger run.

2. **MEDIUM — Card 9 (D6), the seam-warning branch: it is also reached through `PullHub`, where the warning is dropped.** `Fabric.Pull` is also called by `PullHub` (`internal/fabricengine/hubpull.go:141`).
   - Each pulled pair is reported as a `HubPullSide`, which has only `Outcome` and `Detail` (`hubpull.go:52-58`).
   - Card 9 adds `PullResult.SeamWarning` but never copies it into the hub-pull side. A foreground hub pull therefore leaves the warning in the log only.
   - D6 asks that the move's result carry it.

3. **MEDIUM — Card 18 (D2), the "hub with no Bolt checkout" refusal branch: no row reaches it.** The pre-run refuses two different states: a cwd outside a hub's worktree, and a hub with no Bolt checkout.
   - `TestHeddleCLI`'s only row for this runs from "a temp directory outside any hub". That fails earlier, at `lymcwd.Resolve`.
   - The no-Bolt branch, which is a git worktree that resolves but whose `BoltDir` is absent, has no row.

4. **LOW — Card 12, `setAsideTag`'s empty-slug fallback is unreachable through the real board replayer.** Every finding `boardengine` reports carries a non-empty entry: a slug, `prime`, or a design-file name (`internal/boardengine/replay.go` `lost`/`refuse` call sites, and `changePrimeOwner` reports under `prime`).
   - So `TestBoltReplayers`'s row "a ledger loss with an empty entry tags `set-aside/replayer/board/<short sha>`" can only be reached by a fake run ledger named `board`.
   - The card neither lists nor describes that fake. It should name the fake, or treat the fallback as a fail-loud wiring guard.

5. **LOW — Card 12, the `ErrReplayerMissing` guard now covers more paths than before.** `replayBolt` would now fail with no run ledger registered even for replays that never needed one: commits that are only cherry-picked, or only seed commits.
   - Today the seam is consulted only inside the by-entry loop (`replay.go:814`). `localBoardEntries` returns before calling `InstalledBoardReplay` when no board slug changed (`replay.go:662-668`).
   - A test binary that does not call `boardkit.InstallReplay` and replays a stencil-only diverged Bolt would newly fail.
   - The card's Shared Decision gives the reason (a replay without a ledger cannot set aside ended runs), but it does not name the cherry-pick-only path it now refuses.

Tool calls beyond my context:
- Read `hubpull.go` lines 60-120 and grepped `hubforge`, the `Topology.Add` callers and the `Pull` callers, to trace every entry into the D6 fire sites.
- Read `battencli/wire.go` around the Worktree-Create claim, listed `WireJunctions` test files without `hubforge`, and checked the locks `Bolt.Pull` takes, to see whether any Add or render path holds the Bolt write lock.
- Read `bolt.go` lines 255-266 and grepped `gitrepo` for rebase, to check whether a Bolt push replaces `heddle.json` without the write lock.
- Grepped the lock use of `Bolt.Push` and `retryBoltPush`, which confirmed the push's retry replays under the Bolt write lock, so I report no finding there.
