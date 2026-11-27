```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: for each finding fixed as proposed, find the case where the proposal itself is wrong.

1. **MEDIUM**: card 5's sentence "a pair ahead only by held commits reads level and its pusher exits keeping the entry", and the F4 rows in cards 20 and 21. The proposal makes only `NewDeps`' `Ahead` aware of the hold. The starter still counts held commits as owed, so a held-only entry turns owed again on every dobby tick.
   - Evidence: `internal/runpusher/start.go` fills the starter's `ahead` with `fabricengine.RunCommitsAhead(t.Location)`, which card 8 does not make hold-aware.
   - The dobby restarts pair pushers through `runpusher.StartIfAhead` (`internal/battencli/dobbyreads.go:512`, `internal/beamcli/dobbydeps.go:187`).
   - Card 6 says `starter.start` with `ifAhead` "writes the target's ledger entry when the target is ahead".
   - Concrete case: a pair holding a dropped landing's `HeldRecords`. Each tick, `StartIfAhead` reads it ahead, writes `FirstOwed`, which makes `Entry.Owed` true, and spawns a pusher. That pusher reads level and clears `FirstOwed`, and the next tick does it all again.
   - So the owed period, the all-clear and the bar minutes keep flipping, and a pusher spawns on every tick: the no-end loop F4 set out to remove.

2. **MEDIUM**: card 19's `LandingStands` trailer clause. Unlike card 11's re-record, it has no exclusion for a dropped landing. That contradicts card 19's own claim that "a later `lym fabric remove` pass[es] the check".
   - Concrete case: a landing dropped by step 2 "with other local commits on the landing commit". No reset is made, so the `Lym-Landing` commit stays past origin on the local parent with no landing recording it.
   - `LandingStands` then reads true, so `lym fabric remove` is refused. So is the pair removal on a hand-over through `HandOverOwnedElsewhere`, which also makes no reset in this case.
   - Card 11 skips such a commit because "whose pair entry holds `HeldRecords`". Card 19's read needs the same exclusion.

3. **MEDIUM**: card 4's F19 sentence, "when a write adds the first owed entry while no clear time stands". This covers only a hub that never cleared.
   - Concrete case: the last owed entry goes at 10:00, so `clear.json` holds 10:00. A new owed entry is added at 13:00. Because a clear time stands, `Update` writes nothing new.
   - Card 21's `SyncStatus` then shows `sync: 180m` in the first second of the new owed period, though the hub had no owed target just before.
   - The rule that holds for both cases: record `now` when a write takes the hub from no `Owed` entry to one.

4. **LOW**: card 6's F3 sentence, "It keeps every entry holding landings". When a pair's worktree is gone by a route outside lym, its landing-holding entry stays forever with nothing to clear it.
   - Concrete case: a beam pair's worktree deleted by hand while its key holds a child's landing. The entry is never pruned, and the beam's pusher cannot push from a worktree that is gone.
   - `restartHubPushers` hands only refused targets to `SettleRefused`. So the entry stays owed, the all-clear never posts, and no refusal names a way forward.

Not findings:
- F10: `PushBeam`'s only callers are `pr.go:81` and `pr.go:175`, so the refusal reaches only `lym beam pr`.
- F14: `Fabric.commit` takes the weft write lock, not `fabric.push.lock` (`internal/fabricengine/commit.go:220`), so `landLocally` holding the push lock across the commit cannot deadlock.

Tool calls:
- Grepped `start.go`, the `StartIfAhead` callers and the `PushBeam` callers, because my context lacked the starter's ahead read and who calls `PushBeam`.
- Read `pendingflags.go`, the `Fabric.Commit` lock use and the users of `weftPushLockFile`, because my context lacked whether the commit path takes the push lock.
