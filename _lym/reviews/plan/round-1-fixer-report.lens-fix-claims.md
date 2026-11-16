```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every claim the fix added or rewrote against the code at HEAD. Two do not hold.

**1. [MEDIUM] Card 7's bound on `resetUnnamedTip` (fix F7) is false for one path.**
- **Changed text:** card 7, the guard paragraph: "It is bounded in three ways: it runs only on a task pair's rejected code-side push, which records-first runs only after the run's own records side landed, so a machine a takeover moved never reaches it".
- **Summary:** a code-side push that is not records-first also reaches the guard. Finalize's parent push is one.
- **Evidence:**
  - In `pushclassified.go:129-131`, any code-side `pushClassified` call that passes no `retryRecords` falls back to `job.repushOverOriginContained`, which is where card 7 puts `resetUnnamedTip`.
  - `PushWarpRebaseFreeAt` (`spawn.go:211`) makes such a call. It pushes only the code side, with no records side before it.
  - `Fabric.PushBranch` (`fabric.go:176`) calls it, and `landingshed/finalize.go:787` calls `parentHandle.PushBranch` for the Finalize parent push.
  - When the parent pair is itself a task pair, a rejected parent push reaches the guard with no records side of its own landed first.
- **Consequence:** the guarantee "a machine a takeover moved never reaches it" does not cover that path. Either the bound needs restating, or `resetUnnamedTip` must be limited to callers that push records first.

**2. [MEDIUM] Card 8's `ErrPushLockBusy` sentence (fix F15) contradicts card 4.**
- **Changed text:** card 8 Intent: "…and the holder lands the flagged sides on release."
- **Summary:** after card 4, the push-lock holder does not land a side it does not itself push.
- **Evidence:**
  - Card 4 says "`coalesceFlagged` no longer respawns a flag of a side the holder does not owe. Such a flag stays on disk for its worktree's next push".
  - Card 4's own `TestPushLock_BoundedWaitGivesUpAndUnboundedWaitBlocks` row pins exactly this: a code flag left by a pair push stays unlanded when the holder pushes records only.
  - The doc on `coalesceFlagged` (`pendingflags.go:122-131`) shows today's respawn is what card 4 removes.
- **Consequence:** a busy pair push behind a holder that pushes records only leaves its code side flagged, and nothing lands it. The seam's own owed retry is what eventually lands it, not the holder.

**Claims that hold:**
- `pushClassified` returns nil for an ended branch (`:423`), and `pushJob.deferral` writes a `deferred` record and returns nil (`pushclassified.go:296-298`). Both back card 3.
- `cancel.go:370-379` commits the intent before `EndForCancel` on every path, and `CommitRecord` is wired to `commitCancelRecord` (`cancelcmd.go:306-307`). Both back card 9.
- The cancel's end-only output names the re-run (`cancel.go:415`).
- `followup.go:107` spawns `SpawnStatusPush`, as the overview now says.
- The five test functions and line numbers added to card 8 match.
- fabriccli already imports pairteardown (`fabric.go`, `remove.go`).
- `OriginRunStatus` reads the pair's records branches through `pairBranchNames`, which is the premise of F8.
- `destroy.go` is the chokepoint, and its executors check in order (`PATTERN-fabric-boundary.md:95-97`).

**Tool calls:**
1. Grepped the callers of `repushOverOriginContained`, the `pushLockBusyError` sites and the `CommitRecord` wiring, because my context lacked those call sites.
2. Read the call sites of `pushClassified` and `pushSideDetached`, `pushanchored.go:140-175` and `cancelcmd.go:300-315`, to find every code-side path into the guard and to check the cancel wiring.
3. Read `spawn.go:190-215` and the doc on `coalesceFlagged`, to see what `PushWarpRebaseFreeAt` is and what the holder does with flags.
4. Grepped the callers of `PushWarpRebaseFreeAt`, to trace that path to a production caller.
5. Read `fabric.go:168-177` and grepped `.PushWarpRebaseFree(`, to follow the wrapper method.
6. Grepped the callers of `.PushBranch(`, which found the Finalize parent push.
