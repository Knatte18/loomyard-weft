# Card 12 — bolt-replayer-registry

**Delete:**
- `//internal/fabricengine/boardreplay.go`
- `internal/fabricengine#boardFileName`
- `internal/fabricengine#boardOverviewFile`
- `internal/fabricengine#boardProjectPrefix`
- `internal/fabricengine#boardDesignPrefix`
- `internal/fabricengine#boardDesignSuffix`
- `internal/fabricengine#boardEntryDone`
- `internal/fabricengine#boardDesignSlug`
- `internal/fabricengine#isBoardOverview`
- `internal/fabricengine#isBoardPath`
- `internal/fabricengine#boardEntriesBySlug`
- `internal/fabricengine#boardCommitSlugs`
- `internal/fabricengine#localBoardEntries`
- `internal/fabricengine#hubLocationOfBoard`
- `internal/fabricengine#boardStep`
- `internal/fabricengine#replayBoardCommit`
- `//internal/fabricengine/boardoverview_test.go`
- `internal/boardengine#FabricReplay.Entries`

**Create:**
- `plan:internal/fabricengine#BoltReplayer` -> `type BoltReplayer interface`
- `plan:internal/fabricengine#RunLedger` -> `type RunLedger interface`
- `plan:internal/fabricengine#LedgerRun` -> `type LedgerRun struct`
- `plan:internal/fabricengine#ReplayFile` -> `type ReplayFile struct`
- `plan:internal/fabricengine#ReplayRequest` -> `type ReplayRequest struct`
- `plan:internal/fabricengine#ReplayResult` -> `type ReplayResult struct`
- `plan:internal/fabricengine#ReplayFinding` -> `type ReplayFinding struct`
- `plan:internal/fabricengine#ErrReplayerMissing` -> `var ErrReplayerMissing = errors.New("fabricengine: no Bolt replayer registered")`
- `plan:internal/fabricengine#boltReplayerSet` -> `var boltReplayerSet atomic.Pointer[[]BoltReplayer]`
- `plan:internal/fabricengine#RegisterBoltReplayer` -> `func RegisterBoltReplayer(r BoltReplayer) (restore func())`
- `plan:internal/fabricengine#registeredReplayers` -> `func registeredReplayers() []BoltReplayer`
- `plan:internal/fabricengine#claimantOf` -> `func claimantOf(path string) (BoltReplayer, bool)`
- `plan:internal/fabricengine#runLedger` -> `func runLedger() (RunLedger, string, error)`
- `plan:internal/fabricengine#ledgerRunsAt` -> `func ledgerRunsAt(repo *gitrepo.Repo, ledger RunLedger, rev string) ([]LedgerRun, error)`
- `plan:internal/fabricengine#commitLedgerRuns` -> `func commitLedgerRuns(repo *gitrepo.Repo, ledger RunLedger, sha, parent string, paths []string) ([]string, error)`
- `plan:internal/fabricengine#localLedgerRuns` -> `func localLedgerRuns(repo *gitrepo.Repo, head string, ahead []string) (slugs []string, created map[string]string, err error)`
- `plan:internal/fabricengine#hubLocationOfBolt` -> `func hubLocationOfBolt(boltDir string) (*lymcwd.Location, error)`
- `plan:internal/fabricengine#boltStep` -> `type boltStep struct`
- `plan:internal/fabricengine#replayClaimedCommit` -> `func replayClaimedCommit(rec *Mutations, repo *gitrepo.Repo, boltDir, mergeBase string, baseline []string, step boltStep, live EndedSet, outcome *ReplayOutcome) error`
- `plan:internal/fabricengine#WithoutBoltReplayersForTest` -> `func WithoutBoltReplayersForTest() (restore func())`
- `plan:internal/fabricengine_test#TestBoltReplayers` -> `func TestBoltReplayers(t *testing.T)`
- `plan:internal/fabricengine_test#fakeReplayer` -> `type fakeReplayer struct`
- `plan:internal/fabricengine_test#emptyEntryLedger` -> `type emptyEntryLedger struct`
- `plan:internal/fabricengine#TestBoltReplay_NamesNoBoardFile` -> `func TestBoltReplay_NamesNoBoardFile(t *testing.T)`
- `plan:internal/boardengine#FabricReplay.Name` -> `func (FabricReplay) Name() string`
- `plan:internal/boardengine#FabricReplay.Claims` -> `func (FabricReplay) Claims(path string) bool`
- `plan:internal/boardengine#FabricReplay.File` -> `func (FabricReplay) File() string`
- `plan:internal/boardengine#FabricReplay.Runs` -> `func (FabricReplay) Runs(data []byte) ([]fabricengine.LedgerRun, error)`
- `plan:internal/boardengine#FabricReplay.CommitRuns` -> `func (FabricReplay) CommitRuns(before, after []byte, paths []string) ([]string, error)`
- `plan:internal/boardengine#boardDesignSlug` -> `func boardDesignSlug(path string) (string, bool)`
- `plan:internal/boardengine#isBoardOverview` -> `func isBoardOverview(path string) bool`
- `plan:internal/boardengine#entriesBySlug` -> `func entriesBySlug(data []byte) (map[string]string, error)`
- `plan:internal/boardengine#TestFabricReplay_Ledger` -> `func TestFabricReplay_Ledger(t *testing.T)`

**Edit:**
- `internal/fabricengine#ReplayOutcome`
- `internal/fabricengine#readEndedSet`
- `internal/fabricengine#createdStampMatches` -> `func createdStampMatches(run LedgerRun, stamp string) bool`
- `internal/fabricengine#setAsideTag`
- `internal/fabricengine#replayBolt`
- `internal/fabricengine#Bolt.endedByOrigin`
- `internal/fabricengine#Bolt.logReplay`
- `internal/fabricengine#Fabric.weftEndedByOrigin`
- `internal/fabricengine#goneEntryEnded`
- `internal/fabricengine#readOriginEnded`
- `internal/fabricengine#addLocalCreatedStamps`
- `internal/fabricengine#OwnerReader`
- `internal/fabricengine#readOriginOwners`
- `internal/fabricengine/doc.go`
- `internal/fabricengine_test#TestRecordsReplay`
- `internal/fabricengine_test#scribbleThenFailReplay`
- `internal/fabricengine_test#scribbleThenFailReplay.Apply`
- `internal/boardengine#FabricReplay`
- `internal/boardengine#FabricReplay.Apply` -> `func (FabricReplay) Apply(req fabricengine.ReplayRequest) (fabricengine.ReplayResult, error)`
- `internal/boardengine#replayRun`
- `internal/boardengine#replayRun.lost`
- `internal/boardengine#replayRun.refuse`
- `internal/boardengine#replayRun.applyDesignFile` -> `func (r *replayRun) applyDesignFile(boardDir string, file fabricengine.ReplayFile) (bool, error)`
- `internal/boardengine/doc.go`
- `internal/boardengine#replayRow`
- `internal/boardengine#TestFabricReplay`
- `internal/boardengine#TestSetOwners`
- `internal/testkit/boardkit#InstallReplay`
- `cmd/lym#newRoot`

**Uses:**
- `internal/fabricengine#ReplayBoltForTest`
- `internal/fabricengine#ReadEndedSetForTest`
- `internal/fabricengine#ReconcileOrigin`
- `internal/fabricengine#blobAt`
- `internal/fabricengine#restoreReplayTip`
- `internal/fabricengine_test#newRecordsFixture`

**Intent:** Replace the board-tailored replay seam with a general registry of Bolt replayers (D7), so fabric's replay, ended set and owner read hold no board file name, board status or board-named type.

The registry, in a new `internal/fabricengine/boltreplay.go` that replaces `boardreplay.go` (the rest stays in `replay.go`):

- `BoltReplayer` has `Name() string`; `Claims(path string) bool`, for a Bolt-root-relative slash path; and `Apply(ReplayRequest) (ReplayResult, error)`, which re-applies, by entry, a local commit's changes to its own files onto the Bolt checkout at origin's tip under its caller's locks.
  `ReplayRequest` is `Dir` and `Files []ReplayFile`, each `ReplayFile` being `Path` and the bytes at the merge base, before the commit and after it (`Base`, `Before`, `After`, nil when absent).
  `ReplayResult` is `Empty`, `Lost` and `Refused`, and `ReplayFinding` is `Entry`, `Field`, `Reason`, `Local`: `BoardReplay`'s reporting shapes under general names.
- `RunLedger`, which one replayer may also implement, has `File() string`, the Bolt file holding the runs; `Runs(data []byte) ([]LedgerRun, error)`, each `LedgerRun` being `Slug`, `Ended` and `Created`; and `CommitRuns(before, after []byte, paths []string) ([]string, error)`, the runs a commit of its paths touches, given its ledger file before and after the commit.
- `RegisterBoltReplayer` adds a replayer, replacing one of the same name, and returns a function that puts back only what that name held before (the replaced replayer, or no replayer of that name), leaving every other name's registration as it then stands; it is atomic, as the old install was, so parallel tests may share one registration and a restore of one name never drops another test's registration of another name.
  `claimantOf(path)` returns the registered replayer claiming a path; `runLedger()` returns the registered run ledger and its replayer name, or `ErrReplayerMissing` wrapped with the way forward: the lym root pre-run registers the replayers, a test registers what it needs (`boardkit.InstallReplay` registers the board's).
- `WithoutBoltReplayersForTest` (in `export_test.go`) empties the registry and returns its restore, for the non-parallel missing-registration rows.

`replay.go`:

- `readEndedSet` reads the run ledger's file at origin's tip and in its history through `ledgerRunsAt`, and a run is ended when the ledger says `Ended` (in place of the `done` status) or is pruned as today; `createdStampMatches` takes a `LedgerRun`.
  No run ledger registered fails it with `ErrReplayerMissing`, and so does `readOriginOwners`.
- `replayBolt` walks the local commits as today: a seed commit is dropped; a commit whose every path some registered replayer claims becomes a `boltStep` replayed by `replayClaimedCommit`; any other commit is cherry-picked.
  It reads the run ledger only once some commit is claimed, as today's code consults the board seam only inside its by-entry loop and `localBoardEntries` only when a board slug changed; a replay of cherry-picked and seed commits alone needs no registration, and a claimed commit with no run ledger registered fails the replay with `ErrReplayerMissing`, the starting tip restored.
  For a claimed commit it reads the ledger's runs through `commitLedgerRuns`; the ended-run rule applies only when that list holds at least one run, as today's `len(slugs) > 0` check does, so a commit touching only `heddle.json` or only an overview file is never set aside by it.
  When every run in a non-empty list is in the live ended set, the commit is set aside whole if it touches no other replayer's path, as today, and otherwise its ledger part is skipped, the original is tagged under the first run's slug, and the other replayers still apply theirs.
- `replayClaimedCommit` groups the commit's paths by claiming replayer, builds one `ReplayRequest` per replayer from the merge base, the parent and the commit, calls each `Apply`, then stages and commits what they kept as one commit with the original message.
  A loss or refusal tags the original: under the finding's entry when it comes from the run ledger (the run's slug), else `set-aside/replayer/<name>/<short sha>`.
  `setAsideTag`'s empty-slug fallback no longer names the board: a ledger set-aside with no slug takes `replayer/<ledger name>`, a shape no `set-aside/<slug>/...` takes since a slug holds no slash.
  The tip restore on a failure works as today.
- `localLedgerRuns` replaces `localBoardEntries` through `commitLedgerRuns` and `Runs`; `hubLocationOfBolt` replaces `hubLocationOfBoard`; `ReplayOutcome.Lost` and `Refused` hold `ReplayFinding`; `Bolt.logReplay` says "Bolt" where it said "board".
- The other ended-set and owner readers (`Bolt.endedByOrigin`, `Fabric.weftEndedByOrigin`, `goneEntryEnded`, `readOriginEnded`, `addLocalCreatedStamps`, `readOriginOwners`) read the ledger's file and `Runs` instead of `board.json` and `Entries`; `OwnerReader`'s byte argument is renamed from `boardJSON` to `ledger`, its doc naming the run ledger's file, with its type unchanged.
- `doc.go`'s replay, ended-set and owner sections describe the registry, the run ledger and the tag shapes.

`boardengine`:

- `FabricReplay` is the board's replayer named `board` and the run ledger: `Claims` holds `board.json`, `README.md`, `project-<slug>.md` and `design-<slug>.md` at the Bolt root, through `isBoardOverview` and `boardDesignSlug` moved here from fabric; `File` is `board.json`; `Runs` is the old `Entries` with `Ended` set for a `done` entry; `CommitRuns` is the old `boardCommitSlugs`, over `entriesBySlug`, with an overview file naming no run.
- `Apply` reads `board.json` and design files out of `req.Files` by path and is otherwise as before; `replayRun`, `lost`, `refuse` and `applyDesignFile` take the new types.
- `boardengine/doc.go`'s replay section names the replayer registration and the run ledger, and that the by-entry merge is `internal/entrymerge`'s.
- `boardkit.InstallReplay`, `TestSetOwners`'s install and `newRoot`'s pre-run call `fabricengine.RegisterBoltReplayer(boardengine.FabricReplay{})`.

Tests:

- `TestFabricReplay` and `replayRow` move to `ReplayFile`, `ReplayFinding` and `Runs` with every row's expectation unchanged; its `Entries` row asserts `Runs`, with `Ended` true for the `done` entry.
- `TestFabricReplay_Ledger` (tier0) carries `TestIsBoardOverview`'s rows as `Claims` rows, plus `board.json` and `design-a.md` claimed and `heddle.json`, `notes/x.md` and `sub/README.md` not; and `CommitRuns` rows: an entry changed, added or removed names it, a design file names its entry, an overview file names none.
- `TestRecordsReplay` keeps every row; `scribbleThenFailReplay` embeds `boardengine.FabricReplay`, so it keeps the board's name, claims and ledger, and its `Apply` takes the new request, and the row registers it with `RegisterBoltReplayer` and restores the board's after.
- `TestBoltReplayers` (tier1, not parallel, since it empties and fills the process-wide registry) asserts: with no replayer registered, a diverged Bolt whose local commit touches only a stencil file replays by cherry-pick with no error, while with only `fakeReplayer` registered a claimed `notes/fake.json` commit fails `ReplayBoltForTest` with `ErrReplayerMissing` naming the registration and leaves the tip; with no replayer registered `ReadEndedSetForTest` fails the same way, and `ReconcileOrigin` with an `OwnerOf` seam over a recorded pair fails wrapping it; with the board's replayer registered beside `fakeReplayer`, named `fake`, claiming `notes/fake.json` and reporting a loss, its kept change is committed and the original tagged `set-aside/replayer/fake/<short sha>`; and with `emptyEntryLedger` registered in the board's place, a ledger loss with an empty entry tags `set-aside/replayer/board/<short sha>`.
  `fakeReplayer` is a test type whose `Apply` writes the commit's `notes/fake.json` bytes and reports one loss; `emptyEntryLedger` embeds `boardengine.FabricReplay`, so it keeps the name `board`, its claims and its ledger, and its `Apply` reports one loss with an empty `Entry`, the one way to reach that fallback, since the board's own findings always carry a slug, `prime` or a design file name.
- `TestBoltReplay_NamesNoBoardFile` (tier0, in a new `internal/fabricengine/boltreplay_test.go` with `//go:build tier0`) parses `replay.go` and `boltreplay.go` and fails naming each string literal equal to `board.json` or `README.md`, or opening with `project-` or `design-`, and each top-level declaration whose name holds `board` in any case, so no board file name or board-named replay type returns to the replay, ended-set and owner-read code; it first checks its matcher against planted sources that hold one of each and one that holds none.
  The ended set's `"done"` how-value and a run state `"done"` stay, being the ended set's own vocabulary and batten's, not a board status; the board status leaves as `boardEntryDone`'s delete and the ledger's `Ended`.

**ImpactSummary:** Replaces fabric's board replay seam and its three readers with the registry across fabricengine's replay, Bolt, pull, push and origin-reconcile code, boardengine's replay, boardkit and the root pre-run; every existing replay, ended-set and owner case keeps its outcome.
