```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I checked whether the reviewer's proposals that the fixer carried out as written are themselves sound.

**MEDIUM — F3's proposal leaves the forward push in card 6 step 1 calling a banned token outside `destroy.go`**
- Changed text: card 6 now says "Step 1's reset of origin's code branch … run[s] through an executor in `internal/fabricengine/destroy.go`".
- Problem: step 1 makes two leased moves. Before the reset, `pushOwedCodeTip` "pushes that tip with a lease on origin's code branch … (`Repo.UpdateRemoteBranchLeased`)". Card 6's **Uses:** still lists `internal/gitrepo#Repo.UpdateRemoteBranchLeased`, so that forward push stays a direct call outside `destroy.go`.
- Evidence: `cmd/lym/destructiveguard_test.go:80` bans the raw substring `".UpdateRemoteBranchLeased("` in every scanned package unless an allowlist entry covers it. Its failure text (`:219`) says such a call "must be reached only through internal/fabricengine/destroy.go's gate". As written, card 6 cannot pass the guard.
- Fix needed: both leased moves in step 1 go through the executor, which already exists as `updateRemoteBranch` (`destroy.go:1618`).

**MEDIUM — F2's proposal turns a dirty-replay deferral into an unbounded owed retry that cannot land**
- Changed text:
  - card 3: a deferral is returned as the `*PushFailedError` carrying the `deferred` record.
  - card 8: `IsPushOwed` counts class `deferred` as owed, and the owed bullet now reads "a deferred records side".
  - card 7: `FailedPushesOwedBy` counts class `deferred`.
- Concrete case: a pair's records worktree is dirty when its records push is rejected.
  - `retryWeftPush` gets `ErrReplayDirty` and returns deferred (`internal/fabricengine/pushclassified.go:666-668`).
  - Card 8's seam reads that as owed and retries "until it lands", up to once a minute with no end. Each retry meets the same dirty worktree and defers again.
  - `ReadRun` reports the push owed, and card 1's way forward says run `lym fabric push`. That defers the same way, and nothing tells the operator to clean or commit the worktree.
  - The other deferral, a held weft write lock (`pushclassified.go:613-615`), is transient, so only the dirty-replay case is affected.
- Fix needed: the dirty-replay deferral is not owed and halts with its record, or the owed path names cleaning the worktree as the way forward.

**MEDIUM — F11's proposal removes D2's grace bound for a tip whose push keeps failing**
- Changed text: card 12's new bullet: "still missing and the local code branch holds the tip: it pushes the tip through `PushRun` … and gives nothing up … Only a tip this repository lacks is given up after `runfsm.CodeTipGrace`."
- Concrete case: the review's own premise is an owner's code side that failed with a class that is not owed, such as `other` or `not_fast_forward`.
  - Each tick, `PushRun` repeats the same push and fails the same way; the failure is logged and the next tick retries.
  - Because the tip is held locally, the grace never gives it up, so the run waits on `code_tip` indefinitely, and so does `lym batten cancel` (card 7's ImpactSummary).
  - Before the fix, the grace put a limit on this wait. Card 1's `CodeTipGrace` doc says a tip "still missing after it is given up", which no longer holds.
- Fix needed: the held-tip push runs within the grace, and a tip still missing past the grace is given up whatever this machine holds; or the card states the indefinite wait and its way forward.

**LOW — F15's added sentence contradicts card 4**
- Changed text: card 8 says "the holder lands the flagged sides on release".
- Problem: card 4 retires the respawn of flags the holder does not owe ("`coalesceFlagged` no longer respawns a flag of a side the holder does not owe. Such a flag stays on disk for its worktree's next push").
- Effect: a holder pushing a different side or worktree no longer lands the seam's flagged side. The seam's own retry does, so the sentence overstates what the holder does.

Tool calls beyond my context:
- grep of `PushFailedError`'s fields and `Unwrap`, to test F5's proposed struct literal and its `errors.Is` claim (it holds).
- grep for the banned tokens in the destructive guard, to test F3's executor proposal against the guard.
- grep of the guard's token list with line numbers, plus the deferral return sites in `pushclassified.go`, to confirm `UpdateRemoteBranchLeased` is banned.
- `sed` of `retryWeftPush`'s first lines, to see what a records deferral covers (F2).
- Read of `pushclassified.go:620-679`, the rest of `retryWeftPush`, to confirm the dirty-replay deferral (F2).
