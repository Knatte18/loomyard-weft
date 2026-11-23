```yaml
lens: guard-reach
outcome: reported
```

**Per guard: reachable or not**

- Card 1 `OwedUnrecorded` skip in `teardownLockOwedError`/`recordOwedRecordsPush`/`pushJob.fail`: reachable (transport, teardown-lock and deferred rows); every owed-class site in `checkOrigin`/`classifyFailure` goes through `pushJob.fail`, so covered.
- Card 1 `pushJob.deferral` returning an error: reachable (held weft write lock in `retryWeftPush`, `pushclassified.go:844-849`, on a rejected records push).
- Card 1 `ErrEntryEnded` in `recordsSideNotLanded`: reachable for the records side (branch gone on origin, entry ended, tracking ref present; `pushanchored.go:358`). The same state on the code side is unguarded — see F2.
- Card 2 three reads: reachable; "pruned tracking ref reads level" reachable; a branch gone on origin whose tracking ref was not pruned reads ahead — see F5.
- Card 3 `primeTakeovers` gated on empty `runSlug`: reachable from the prime pusher; the guarded replay recurs unguarded on every prime records push naming a run slug — see F1.
- Card 3 `ErrTakeoverPassOwed` on a failed takeover read: reachable (invalid JSON at origin's tip fails `originStatusAtTip` after the fetch; `rejectionRetryKind` with widened `isTakeoverStop` returns it unrecorded).
- Card 3 `withoutFinalizeEntered` keeping `EndedHandedOver`: reachable (Land-row row); set-aside holds since every `battenRunCommitPaths` path lies under `RunDirRel` (`shedrun/paths.go:307-497`), which `runDirectorySlug` checks (`replay.go:384-512`).
- Card 3/4 `PrimeRecords` archive failure returning before the mark: reachable in principle, but no planned row isolates it — see F3.
- Card 4 failed hand-over → `ErrTakeoverPassOwed`; stop on non-`*TakeoversError`: reachable.
- Card 5 start conditions (lock, refused push, pause, hand-over mark, skip flags) and `Run`'s pause-before-guard: reachable over fakes; a pair's hand-over mark occurs since an `InsidePair` hand-over leaves the pair standing.
- Card 6 `pusher --worktree <missing>` past the pre-run: reachable (`resolvePersistentPreRun` skips by name at `loomcli/cli.go:255`, so the row reaches the verb's own resolve error).
- Card 7 `commit-records` envelope `pusher_started`: no source for its value — see F4.
- Card 8 resume start / continue-pass no start / dobby `StartIfAhead`: reachable.
- Card 9 `localHoldsTip` branch: reachable; the settle needs no other guard since `settleCodeTip` acts only on a `code_tip` verdict reason (`battencli/codetipsettle.go:117,144`).
- Card 10 Stuck/handed-over/`ErrEntryEnded`/`Level`-false re-push branches: reachable over fakes; the `Level`-false loop has a non-exiting state — see F2.

**Findings**

- **F1 — BLOCKING · design · card 3 (`03-fabric-prime-takeover.md`, the `retryWeftPush` bullet)** — The prime takeover read is gated on an empty `runSlug`, but prime records pushes that name a slug still replay every batten run's unpushed commits, a taken-over run's included, onto origin over its takeover — breaking D6's "never replayed". Callers naming a slug: batten's kept `Push` for the cancel and re-adopt records (`PushRun(location, location, runID, …)` on the prime, `battencli/commitstatus.go:172`, `readopt.go:99`, `cancelcmd.go:498`), `pushOwedRunCommits` (`pairteardown/handover.go:185`), and the detached coalesce push whose `runSlug` is `req.PauseSlug` (`fabricengine/coalesce.go:150,177`; e.g. cleanup's `spawnStatusPush` with a gate slug). On those paths `takeoverStop` reads only that one run (`pushclassified.go:827-835`) and `weftEndedByOrigin` sets aside only ended runs. Fix: run `primeTakeovers` on every prime records push with a non-zero `here`, whatever the slug, keeping `takeoverStop` for pair records worktrees only.
- **F2 — MEDIUM · design · card 10 (`10-teardown-pushes-first.md`, the `Call` order bullet)** — The `ErrEntryEnded` escape covers only the records side; for a code branch origin deleted whose entry ended, the code side's `checkOrigin` returns proceed false with nil (`pushclassified.go:641-642`), so `Push` returns nil, then `Level` (`TipsOnOrigin`) reads the still-present tracking ref lacking the local code tip and returns false — the teardown re-pushes forever. Fix: `TipsOnOrigin` reads a branch gone on origin with an ended entry as held, or the code side's ended skip surfaces `ErrEntryEnded` too.
- **F3 — MEDIUM · design · card 3 (the `TestHandOverTips` rows) and card 4 (`TestPushRun_HandsOverOnATakeover` archive-failure row)** — No row reaches the new `PrimeRecords` archive failure in isolation: `TestHandOverTips` builds its request with `Pair` set (`handover_integration_test.go:54-55`), so a tag-refusing hook fails the pair's `archiveOwnerSides` first (`fabricengine/handover.go:88-91`), and the "`ErrArchiveFailed`, no mark" assertion passes whether or not the prime-records archive exists; card 4's row has the run's pair standing too. Fix: add a row with `Pair` nil, or a hook that refuses only the prime records archive tag.
- **F4 — LOW · consistency · card 7 (`07-transitions-start-pusher.md`, the `commitRecordsVerb` bullet)** — The envelope reports `pusher_started`, but `StartPush` is `func()` with no result, so the verb cannot know whether a pusher started (`runpusher.Start` returns `(bool, error)`). Fix: make `StartPush` return the started bool, or drop the key.
- **F5 — LOW · design · card 2 (`02-fabric-pusher-reads.md`, `RunCommitsAhead`) with card 8** — A records branch gone on origin with an ended entry whose tracking ref was not pruned reads ahead; its pusher gets the `ErrEntryEnded` error, neither owed nor refused, writes no record and exits (`pushanchored.go:358`), so every dobby tick starts another pusher that fetches the board and exits, until teardown. Fix: read a branch origin's ended-check marks gone as level, or have the pusher's exit on `ErrEntryEnded` suppress restarts.

Tool calls:
1. Grep for `PairPush`/`Here:`/`retryWeftPush` callers and `OriginCodeTip` callers — to find prime pushes passing a slug and other code-tip readers; manifest lacked callers.
2. Read `coalesce.go` 150-190 and `codetipsettle.go` 35-60 — the detached push's runSlug/here and the code-tip settle reader were not in the manifest.
3. Read `settleCodeTip` body and coalesce callers — to check whether the settle acts without a code-tip verdict and where detached pushes get `here`.
4. Read `teardownlock.go` 115-145 and fabriccli `ThisMachine` uses — to confirm detached pushes pass a non-zero `here`.
5. Read `runDirectorySlug` use in `replay.go` and `resolvePersistentPreRun` — the set-aside rule card 3 relies on and the pre-run skip card 6's row passes.
6. Grep `shedrun` path functions — to confirm every batten commit path lies under `RunDirRel`.
