```yaml
lens: guard-reach
outcome: reported
```

## guard-reach: per-guard list

- **Card 2.** Network re-step with no cap: reachable from `goDriver.settleStop`. The zero `NetworkWaitCeiling` falling back to 15 s is reachable only from a hand-built `Deps`, since `goDriverDepsFor` always sets it; that is a fail-safe, not a defect. Removing a wait record at driver start is reachable after a killed driver.
- **Card 3.** `failedHalt` writing no note under a held lock: reachable from a Go-driver step. `WaitDied`: reachable after a killed driver. The `Waiting` gap between the failed status and the first record write: reachable.
- **Card 5.**
  - `SingleAttempt`: reachable through `pusherDeps`.
  - Skipping the wait once on `PushNow`: reachable from the spawns.
  - "Level, the pass removes the entry unless it holds landings or `HeldRecords`": **not reachable** (finding 4).
- **Card 6.**
  - The starter's `ifAhead` write and removal, and its refusal skip: reachable.
  - `PruneLedger` dropping an entry whose worktree is gone: reachable, but it recurs on entries that hold landings, unguarded (finding 3).
- **Card 7.** No spawn under `FABRIC_SKIP_*`: reachable.
- **Card 9.**
  - `RecordsHold` in `PushRun`: reachable.
  - The guarded site recurs unguarded in `lym fabric push` (finding 1).
  - `CodeCeiling`: reachable, but the parent push it should let through cannot pass a ceiling (finding 2), and it also caps `PushBeam` (finding 5).
- **Cards 10 and 11.**
  - `held_back` for a parent or a run owned elsewhere: reachable.
  - The "landing commit no entry records" refusal: reachable, but its way forward is dead in most of the states that reach it (finding 6).
  - Step 2's drop with the reset or the rebase refusal: reachable.
- **Card 12.** `ParentAheadByOwed` (second landing), the owner-check `git-transport` stop and `syncParent`'s transport stop: reachable. The crash resume is reachable: `syncParent` returns early on `held` (`internal/landingshed/finalize.go:590`), so `pushParent` runs.
- **Card 13.** The check before the lock and the re-check under the lock: reachable.
- **Card 14.** The base-file guard: reachable.
- **Card 16.** The owed-tip level read and Land's `git-transport` stop: reachable.
- **Card 17.** The held branch and the owed lock-busy branch: reachable.
- **Card 18.** `unmadeOriginCommit`: reachable through `resetUnnamedTip`.
- **Card 19.**
  - `Run` and `RemovePair` refusing through `lym fabric remove` and the dobby: reachable.
  - `HandOver` skipping the removal: reachable when a records push meets a takeover.
  - `Level` false while a landing stands: reachable.
  - `tearDownLeftover`'s guard: reachable.
  - `EndForCancel`'s guard: mostly unreachable from `lym batten cancel`, and where reached it comes after the cancel's own mutation (finding 7).
  - The unconditional ledger removal after `RemovePair`: the guarded site recurs unguarded (finding 3).
- **Card 21.** `sync: ?`: reachable.

## Findings

1. **BLOCKING, design — card 9 (`09-records-hold.md`).** The records hold sits only in `PushRun`. `lym fabric push` in a task pair calls `fabricengine.PushPairAnchored` directly with no `RunSlug` and no `RecordsHold` (`internal/fabriccli/weft_verbs.go:314`). The verb's detached-child path, through `fabricengine.DetachedPush` (`weft_verbs.go:280`), is the same. Either sends a held records commit to origin while the landing entry stands, against D1's last sentence and D5's "the hold binds every push of the run's pair records". The card's Bound covers only a beam's worktree.
   - Fix: set `RecordsHold` inside `PushPairAnchored` from `pushledger.HeldRecordsCommit` for a task pair's location, or route the verb through `PushRun`. Add a `TestPushRun_StopsShortOfAHeldRecordsCommit` row driving `lym fabric push`.

2. **BLOCKING, design — cards 9 and 11 (`11-parent-pusher.md`, step 5).** Card 11 has the beam parent push run "the beam pair's `PushRun` with that tip as its code ceiling". No card adds a ceiling input to `pairteardown.RunPush` (`internal/pairteardown/handover.go:22-37`). Card 9 has `PushRun` derive `CodeCeiling` itself as the first parent of the oldest landing's tip. So the parent push is always capped below every landing commit, origin's beam branch never takes a landing, and step 1 never settles a beam landing.
   - Fix: card 9 adds a `CodeCeiling` field to `RunPush` that `PushRun` passes in place of its derived ceiling. Card 11 lists `internal/pairteardown#RunPush` and `#PushRun` and sets the field to the judged tip.

3. **BLOCKING, design — cards 6, 19 and 4 (`PruneLedger`, `Teardown.RemovePair`).** D3: "A parent target's entry stays while any of its landing entries stands."
   - Card 19's `landingStands` reads only the removed run's own landings. After a successful removal, `RemovePair` drops the pair target's entry unconditionally. Removing a beam pair, through `lym fabric remove <beam> --force` or the beam's own teardown, therefore deletes the child landings on the beam's key. That releases each child's hold (`HeldRecordsCommit` reads them gone), loses its pending `done` write and lets the child's teardown pass.
   - Card 6's `PruneLedger` drops any entry whose `Worktree` is gone, with no landing exception.
   - No card says who writes `Worktree` on the landing-key entry. Card 11's `RecordLanding` sets only the landing and the mark, so an empty `Worktree` may read as gone and be pruned.
   - Fix: the refusal in `RemovePair` also covers landings held on the removed pair's own key, and `PruneLedger` keeps every entry holding landings. `RecordLanding` writes `Worktree` and `Slug` on the entry, or `PruneLedger` skips the landing and board keys.

4. **MEDIUM, design — card 5 (`pushUntilLevel`), with card 6's `Ahead`.** The "level pass keeping the entry while it holds a landing or `HeldRecords`" branch cannot occur.
   - Card 6 makes `Ahead` true whenever the entry holds a landing.
   - A pair holding `HeldRecords` stays ahead by the held commit itself, since `RunCommitsAhead` counts it and the push stops short at `<hold>^`.
   - So the pass never reads level: each push "lands" as a no-op and the pusher retries once per interval for as long as the hold stands. For a dropped landing owned here, that lasts until `lym fabric remove`.
   - The `TestRun` row "a level pass ... keeping it while it holds a landing" pins an unreachable state.
   - Fix: `Ahead` for a held pair reads only commits up to `<hold>^`, and for a landing-holding target the pass waits on the push-now mark rather than re-pushing. Replace the row with that state.

5. **MEDIUM, consistency — card 9 (`PushRun`'s `CodeCeiling`).** `lym beam pr` pushes through `PushBeam` (`internal/beamcli/deps.go:366-378`) and then uses `pair.HeadSHA()` as the pull request's head (`internal/beamcli/pr.go:81`, `pr.go:175`). With landings standing on the beam, card 9 caps that push below the landing commits, so the PR's recorded head is a commit origin lacks. No card states what `lym beam pr` or `ready` does while landings stand on the beam.
   - Fix: card 9 or 13 names `PushBeam` and has `lym beam pr` stop as `git-transport`, or refuse, while the beam's target holds landings, with a `TestOpen`-style row.

6. **BLOCKING, design — card 11 (step 4, the unrecorded-landing refusal).** The way forward `lym loom resume` in that run's task worktree works only while the run is still at Finalize. `internal/loomcli/resume.go:88` answers "the run is done; there is nothing to resume" for a done run. The refusal is also reached after Finalize returned done:
   - after the ledger is lost (`.lym` is a cache per PATTERN-lym-files);
   - after finding 3's removals drop an entry;
   - after a dropped landing with other local commits on it, where step 2's rebase refusal and this one both match the same branch's single record.

   Under PATTERN-refusal-way-forward the named way forward must work, and a test must reach it.
   - Fix: the way forward names a step that records or drops the landing for a done run, such as the rebase that drops the commit, or `git push` after `lym adopt`. Step 4 skips a commit whose run's landing step 2 dropped. Add a `TestParentPush` row for a done run.

7. **LOW, design — card 19 (the `EndForCancel` guard).**
   - From `lym batten cancel`, the `ChildReachedParent` refusal (`internal/battencli/cancel.go:337-342`, through `childParentHolds`) stops a run past its merge first, with no mutation, so the guard is rarely reached.
   - Where it is reached, for example with the child status unreadable, `WriteIntent` and `CommitRecord` (`cancel.go:380-390`) have already committed the cancel intent and spawned its push, against D13's "refuses before any mutation".
   - The new `TestEndForCancel` row pins the guard alone, not cancel's real order.
   - Fix: card 19 has the cancel verb read `landingStands` before `WriteIntent`, with a `battencli` cancel row.

Tool calls:
- grep of `internal/loomcli` for the resume answer on a done run, and of `landingdeps.go` for the parent-holds seam: the manifest held neither the resume path nor that seam's wiring.
- grep of `internal/battencli/cancel.go` for where `EndForCancel`, the claim clear and the records push sit: the manifest held only `cancelcmd.go`'s deps.
- read of `internal/battencli/cancel.go:330-400`: to see what cancel mutates before `EndForCancel`.
- grep of `internal/battencli/cancelcmd.go` for `ChildReachedParent`'s wiring: to judge whether cancel refuses a landed run before the guard.
