```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens review of the plan (cards 1–11): changes forced outside the members the cards name.

**BLOCKING — Card 7 (batten `commitStatusDeps` loses `Push`, and `pushRecordsOrOwe` is deleted): the cancel's record commit is left uncovered.**
- `internal/battencli#commitCancelRecord` (`internal/battencli/cancelcmd.go:490-499`) ends with `return true, pushRecordsOrOwe(deps, slug)`.
- Its production caller passes `battenCommitStatusDeps(location, slug)` (`cancelcmd.go:308`).
- Card 7 deletes `pushRecordsOrOwe`, removes `Push` from `commitStatusDeps` and drops the fill from `battenCommitStatusDeps`. The build breaks at `cancelcmd.go:498`.
- `TestCommitCancelRecord` builds the deps with `Push:` at `internal/battencli/cancel_test.go:1186`, so it fails to compile too.
- The cancel-intent push is a tracked-intent push, which the record lists as Out of scope. It must keep a push of its own: either a push seam beside the struct, or a direct `pairteardown.PushRun` in `commitCancelRecord`.
- Neither `commitCancelRecord` nor `TestCommitCancelRecord` is on any card.

**BLOCKING — Card 7, the same struct change: the batten re-adopt path is unchecked.**
- `internal/battencli/readopt.go:95` builds `battenCommitStatusDeps(location, slug)`. Card 7 says readopt "reads only `Commit`".
- `readopt.go:83` also fills a `SpawnPush` seam, which shows readopt pushes on its own path.
- No card lists or re-reads readopt's deps use against the new struct. Only `Commit` was confirmed, so this is a gap, not a known break.

**MEDIUM — Card 7, test set incomplete: two more tests build the deps with the removed fields.**
- `internal/battencli#TestFreshStartCommitsWorktreeCreateThroughTheSeam` (`predrive_test.go:196`, `Push:`) is listed. `internal/battencli#TestFreshStartSeedCommitFailure` in the same file is not; it drives the same seeded commit through `newPreDriveReceiver` and should be checked.
- `TestCommitCancelRecord` (`cancel_test.go:1186`) is unlisted, as above.

**MEDIUM — Cards 5 and 7: `pattern/PATTERN-daemon-wakeups.md` goes stale.**
- Card 7 removes `internal/statuscommit/statuscommit.go` from `cmd/lym#unboundedWaitFiles`.
- Card 5 adds the `runpusher` file to that list.
- The background's "What counts" list still names "the owed transition push's retry in `internal/statuscommit/statuscommit.go`" (`pattern/PATTERN-daemon-wakeups.md:100`). No card edits it, and the pusher's owed backoff and re-read loop is never listed there.

**MEDIUM — Card 11 covers `docs/overview.md` only partly.**
- It rewrites the `commit-records` line only.
- `docs/overview.md:339` still describes `internal/statuscommit` as "skip while mid-merge, commit hard-errors, push spawn warns".
- The tree listing gains no `internal/runpusher/` line for the new package.

**LOW — Card 7: a test comment goes stale.** `internal/loomcli/wiring_commitstatus_test.go:3` says "The seam's three dispositions live in internal/statuscommit". After card 7 the core has two dispositions (skip, commit) plus a start, and the file is edited without this line being named.

**MEDIUM — Card 5: an anchor change ends every pusher, and no card states it.**
- `internal/fabriccli/anchor.go:468` ends `runproc.RolePush` processes of the dobbies' owners during an anchor change.
- `internal/fabriccli/anchordeps.go:466` (`liveUnsparedChild`) spares RolePush children from its busy wait.
- The pusher registers under `runproc.HubOwner` with `RolePush`, so an anchor change of the prime ends a live pusher mid-backoff. The pusher has become a long-lived process, and neither card 5's Intent nor the anchor-change path says so or bounds it.
- The next commit or dobby tick restarts it. That restart is acceptable, but it should be written into card 5 rather than left unstated.

**LOW — Card 7 Intent: one stated fact is wrong.** It says `prePushCheck`, `guardedCommit` and `guardedPush` "stay for the landing's pushes". `guardedCommit` is also used by `pullRequestRecordCommitter` (`internal/loomcli/prreviewgoto.go:80`, `prfindings.go:344`, `prcontinue.go:206`, via `landingdeps.go:530`). The claim holds in substance, since those callers stay, but it is incomplete.

Tool calls beyond context:
1. Grepped repo-wide for `statuscommit` mentions — to find docs, pattern files and lists naming the package that the pre-read lacked.
2. Grepped non-test callers of `loomCommitStatusDeps`, `guardedPush` and `guardedCommit`, plus `RolePush` readers and `anchordeps.go:450-480` — to find consumers of the removed loom deps and of the pusher's role.
3. Grepped test files that build the changed deps structs — to check the completeness of card 7's test list.
4. Grepped the exact `Push:`, `Admit:` and `Owed:` lines in those test files — to see which unlisted tests break.
5. Grepped battencli for other consumers of `commitStatusDeps` and read `anchor.go:455-472` — to find non-test users of the struct beyond `commitstatus.go`.
6. Read `cancelcmd.go:486-520` and the callers of `battenCommitStatusDeps` — to confirm `commitCancelRecord`'s use of `pushRecordsOrOwe`.
