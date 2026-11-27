```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens review of the plan's 23 cards. Every existing glyph the cards name resolves through `lym quarry resolve`; the defects below are forced edits no card lists.

- **MEDIUM — Card 11, `TestParentPush` handle package.** The handle is `plan:internal/runpusher_test#TestParentPush`, but its Intent has the test pin "`starter.start` spawning only a settle-only pusher". `starter` and `starter.start` are unexported in `internal/runpusher/start.go:41,108`, and today `internal/runpusher#TestStart` drives them from inside the package. An external `runpusher_test` test cannot reach that branch. Evidence: `internal/runpusher/start_test.go:3` (`package runpusher`) against `internal/runpusher/run_test.go:3` (`package runpusher_test`). Fix: move the settle-only spawn row into `internal/runpusher#TestStart`, or declare the test in `internal/runpusher`.

- **MEDIUM — Card 2, `internal/sheddriver/doc.go` listed under Uses but falsified.** `internal/sheddriver/doc.go:50-52` says "The network-failure waits grow with each re-step… Only a failure that outlasts the waits reaches the parent, as a notice". Card 2 removes that hand-back and its notice. The doc file belongs under `**Edit:**` with the paragraph rewritten.

- **MEDIUM — Card 7, two package docs listed under Uses but falsified.**
  - `internal/boardengine/doc.go:352` says the sync "is spawned as `lym board sync --deferrable`".
  - `internal/fabricengine/doc.go:95` says `Commit` "spawns a detached, fire-and-forget push through `SpawnDetachedPush`".

  Card 7 makes both spawns start `lym loom pusher --push-now`. Both doc files belong under `**Edit:**`.

- **MEDIUM — Card 19, `internal/battenshed/doc.go` listed under Uses but falsified.** `internal/battenshed/doc.go:111-116` says the row "pushes the pair between two holds… After a bounded number of pushes… the row is Stuck". Card 19 removes the push rounds and their Stuck. The doc file belongs under `**Edit:**`.

- **MEDIUM — Card 12, the landing's docs listed under Uses or left unlisted, and falsified.**
  - `internal/landingshed/doc.go:45` says "It never pushes or rewrites a parent that is ahead of or diverged from its upstream". Card 12 now lands on a parent that is ahead by owed tips.
  - `internal/landingshed/doc.go:51` contrasts the GitHub merge with "landing by a local push".
  - The `Finalize.Call` doc (`finalize.go:238-270`) still describes the parent push and the protected-branch way forward, and ends "the board task is marked done, the push is a no-op".
  - The file headers `finalize.go:4` ("pushes the parent branch") and `finalize_owner.go:3` (the repair note) describe code the card removes.
  - The `markTaskDone` doc (`finalize.go:804`) says the write comes "once the parent branch and the run's records are on origin".

  None of these is an Edit target. Card 12 lists `doc.go` under Uses, and neither `internal/landingshed#Finalize.Call` nor the two file headers appear at all. Fix: add `internal/landingshed/doc.go` and `internal/landingshed#Finalize.Call` under `**Edit:**`, and state the header rewrites in the Intent.

- **LOW — Card 12, stale `nilLegal` keys.** `internal/testkit/envkit/envkit.go:92-97` holds `"Landing.OriginParentHoldsWork"`, `"Landing.ParentCommitsOriginLacks"`, `"Landing.PushRecords"` and `"Landing.ArchiveParentMerge"`. Card 12 deletes those four fields, but its Intent only adds the three new seams to `nilLegal`. It should also say the four stale keys go.

- **LOW — Cards 4 and 14, new `lymdirs` constants without doc lines.** `internal/lymdirs/doc.go:9-10` documents each hub-level directory name, `PusherDirName` and `HandOverDirName` among them. Cards 4 and 14 add `PushLedgerDirName` and `IgnoreCheckDirName` but list `internal/lymdirs/doc.go` only under Uses. It belongs under `**Extends:**` with one line per name.

- **LOW — Card 19, `internal/runpusher/doc.go` under Uses while the card deletes a documented file.** `internal/runpusher/doc.go:27` documents `PushUntilLanded`, and card 19 deletes `internal/runpusher/pushuntillanded.go`. Card 5 rewrites the Running section, but its Intent keeps `PushUntilLanded` alive until card 19, so any remaining mention goes stale. The doc belongs on card 19 under `**Edit:**`.

- **LOW — Card 21, `bar.go` file comment unlisted.** The file comment at `internal/reedengine/bar.go:1-28` lists every bar option, including "The server-global @lym_slack_state_command holds the job's line". Card 21 adds `@lym_sync_status_command` and a field on line 2 but does not list `internal/reedengine/bar.go#` for that comment. Also, the tier2 test `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` (`bar_integration_test.go:308`) reads line 2, which now gains a field before the check mark. The card should list or `Uses:` it.

- **LOW — Card 7, file comments and dead fields.**
  - The file comments `internal/fabriccli/spawn.go:1-3` ("so the detached child pushes the sides…"), `internal/boardengine/spawn.go:1` and `internal/fabricengine/spawn.go:1-3` describe a push child that no longer exists. Card 7 lists only members.
  - After the change, `statusPushRequest`'s `TeardownSlug`, `ParentLocked` and `PauseSlug`, and `fabricengine#parentLockHeldBy`, feed no reader.
  - The Intent's claim that they "stay for `lym fabric push`'s own flags" is wrong: those flags come from `pushSpawnCommand`, which the card rewrites. The card should either delete them or name their remaining reader.

- **LOW — Card 4, `docs/overview.md` may need the new package.** `docs/overview.md` names `internal/runpusher`, so it likely catalogues packages, and no card adds `internal/pushledger` or the pusher's new behaviour there. I did not open the file, so this is unconfirmed.

- **LOW (outside lens, guard-reach) — Card 19's refusal can break a hand-over.** `RemovePair` refuses while a landing entry stands, and `pairteardown#HandOver` calls `td.RemovePair` (`internal/pairteardown/handover.go:191`). A takeover reached through `PushRun` would then fail at "remove its local pair". That covers the pair pusher, the catch-up and cancel, which run outside card 11's drop path. No card bounds this.

Tool calls I made:
1. A grep for callers and doc mentions of the deleted and changed members. My context had only the fork0 exploration, not those references.
2. A filter of that grep's saved output, because the output was too large to show inline.
3. A `lym quarry resolve` of the cards' existing glyphs (detail only), to check each spelling against the tree.
4. A second `lym quarry resolve` printing only missing glyphs (none), plus an outline of `envkit.go`, because the first call's output did not show which glyphs were missing.
5. Reads of the `envkit`/`shedrecipe` builders, the fabric config test and the package doc lines that the cards list under Uses. My context had not read those doc files.
6. A grep for the `lymdirs` name registrations and for docs that name `internal/runpusher`, to find registries the new package and new directories must join.
