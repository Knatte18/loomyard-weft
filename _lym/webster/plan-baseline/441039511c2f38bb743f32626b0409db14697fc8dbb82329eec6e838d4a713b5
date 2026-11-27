# Card 4 — push-ledger

**Create:**
- `internal/pushledger/doc.go`
- `internal/pushledger/ledger_test.go`
- `internal/pushledger/testmain_test.go`
- `internal/pushledger#Key`
- `internal/pushledger#BoardKey`
- `internal/pushledger#LandingKey`
- `internal/pushledger#RecordsKey`
- `internal/pushledger#Entry`
- `internal/pushledger#Entry.Owed`
- `internal/pushledger#Landing`
- `internal/pushledger#Read`
- `internal/pushledger#Update`
- `internal/pushledger#List`
- `internal/pushledger#LastClear`
- `internal/pushledger#HeldRecordsCommit`
- `internal/pushledger#OwedTips`
- `internal/pushledger#EntryPath`
- `internal/lymdirs#PushLedgerDirName`

**Extends:**
- `internal/lymdirs/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/fabriclayout#HubScratchDir`
- `internal/state#UpdateJSON`
- `internal/state#ReadJSON`
- `internal/lock#AcquireWriteLock`

**Intent:** D3's owed ledger and D5's landing entries, as a leaf package every pusher, landing, teardown and dobby card reads.
`internal/pushledger` imports only `internal/state`, `internal/lock`, `internal/lymdirs`, `internal/fabriclayout` and the standard library,
so `internal/fabricengine`, `internal/boardengine`, `internal/pairteardown`, `internal/runpusher` and `internal/dobbyengine` can all import it without a cycle.
`doc.go` documents the package alone, per PATTERN-code-conventions.
`internal/lymdirs/doc.go` gains the line for `PushLedgerDirName` beside the other hub-level names, and `docs/overview.md`'s package tree gains `internal/pushledger/` with a one-line description beside `internal/runpusher/`.

The ledger is machine-local, under the hub's never-tracked directory: `<fabriclayout.HubScratchDir>/<lymdirs.PushLedgerDirName>/`, per PATTERN-lym-files.
A target's entry is one JSON file, `EntryPath`, its key path-escaped as one file name.
`RecordsKey` keys the prime's and a pair's target by the base name of their records worktree, as `runpusher.LockPath` names its lock;
`BoardKey` is the board's branch and `LandingKey` the landing target of the prime's code branch.
`Entry` holds `Worktree` (the code worktree path, empty for the board), `Slug`, `FirstOwed`, `LastAttempt`, `IntervalS` (the interval in force), `PushNow`, `Landings`, `HeldRecords` and `Dropped`.
`Landing` holds `Run` (the landed run's slug), `Tip` (the landed parent tip), `RecordsCommit` (the run's records commit its pair target holds), `ParentBranch` and `At`.
A `Landing`'s pending `done` write is the entry itself: it stands until the parent target settles or drops it.
`HeldRecords` is the records commit a dropped landing leaves held on the run's own pair target, so the hold outlives the landing until the pair goes (D5).
`Dropped`, on a parent target's entry, maps each run whose landing card 11 dropped to its landing commit, kept while that commit stands on the parent past origin, so no read turns a dropped commit back into a landing once its pair goes.
`Entry.Owed` is true while the entry holds landings or a `FirstOwed`; an entry holding only `HeldRecords` or `Dropped` is held, not owed, since nothing can push it before its pair goes or its commit leaves the parent (D3).

`Update` reads the entry and writes what `mutate` returns, or removes the file when `keep` is false, under one ledger lock, `ledger.lock` in the ledger directory, taken through `internal/lock`.
Under the same lock it keeps the hub file `clear.json`, the time the hub last had no owed target, with `now` passed in so a test injects the clock:
it records `now` when a write leaves no entry `Owed`, and when a write takes the hub from no `Owed` entry to one, so an owed period counts from its first owed entry, on a hub that never cleared as on one that did (D3, D4).
Entries are written through `internal/state`, so a record with an unknown key reads, per PATTERN-tolerant-records.
`List` returns every entry, `LastClear` the hub file's time (zero when none was written), `OwedTips` the `Tip` of each landing of a key in landing order,
and `HeldRecordsCommit` the `RecordsCommit` of the run's oldest standing landing on any key, or else the `HeldRecords` of the entry whose `Slug` is the run.
A missing directory reads as an empty ledger.

`internal/pushledger/ledger_test.go` is tier0, adding under a second to one tier0 run.
`internal/pushledger/testmain_test.go` is untagged support: its `TestMain` runs the package through `tmuxkit.Main`.
`ledger_test.go` pins over `t.TempDir()`: an `Update` that writes and one that removes, the clear time set to the `now` passed when the last owed entry goes and not while another stands,
the clear time set to the `now` of a first owed entry on an empty hub, and to the `now` of a new owed entry hours after an earlier clear, an entry holding only `HeldRecords`, and one holding only `Dropped`, reading not `Owed` and leaving the clear time written,
`OwedTips` and `HeldRecordsCommit` over two landings of one run and one of another, `HeldRecordsCommit` reading a pair entry's `HeldRecords` with no landing standing, an entry file with an unknown key that reads, and an empty hub.

**ImpactSummary:** Adds one directory name to `lymdirs` and one line to its doc and to `docs/overview.md`; the rest is a new package no code reads yet.
