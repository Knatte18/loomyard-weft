# Card 1 — landing-queue-store

**Create:**
- `internal/landingqueue/doc.go`
- `plan:internal/lymdirs#LandingQueueDirName` -> `const LandingQueueDirName = "landing-queue"`
- `plan:internal/landingqueue#Outcome` -> `type Outcome string`
- `plan:internal/landingqueue#OutcomeLanded` -> `const OutcomeLanded Outcome = "landed"`
- `plan:internal/landingqueue#OutcomeConflict` -> `const OutcomeConflict Outcome = "conflict"`
- `plan:internal/landingqueue#OutcomeFailed` -> `const OutcomeFailed Outcome = "failed"`
- `plan:internal/landingqueue#Entry` -> `type Entry struct`
- `plan:internal/landingqueue#MoveMark` -> `type MoveMark struct`
- `plan:internal/landingqueue#MarkMember` -> `type MarkMember struct`
- `plan:internal/landingqueue#BackstopSeams` -> `type BackstopSeams struct`
- `plan:internal/landingqueue#Store` -> `type Store struct`
- `plan:internal/landingqueue#Dir` -> `func Dir(boardDir string) string`
- `plan:internal/landingqueue#Store.Join` -> `func (s *Store) Join(entry Entry) error`
- `plan:internal/landingqueue#Store.Read` -> `func (s *Store) Read(slug string) (Entry, bool, error)`
- `plan:internal/landingqueue#Store.Entries` -> `func (s *Store) Entries() ([]Entry, error)`
- `plan:internal/landingqueue#Store.Live` -> `func (s *Store) Live(entry Entry) bool`
- `plan:internal/landingqueue#Store.SetOutcome` -> `func (s *Store) SetOutcome(slug string, outcome Outcome, commit string) error`
- `plan:internal/landingqueue#Store.Remove` -> `func (s *Store) Remove(slug string) error`
- `plan:internal/landingqueue#Store.Prune` -> `func (s *Store) Prune() ([]string, error)`
- `plan:internal/landingqueue#Store.WriteMark` -> `func (s *Store) WriteMark(mark MoveMark) error`
- `plan:internal/landingqueue#Store.ReadMark` -> `func (s *Store) ReadMark() (MoveMark, bool, error)`
- `plan:internal/landingqueue#Store.RemoveMark` -> `func (s *Store) RemoveMark() error`
- `plan:internal/landingqueue#Store.Backstop` -> `func (s *Store) Backstop(seams BackstopSeams) (int, error)`
- `internal/landingqueue/queue_test.go`

**Uses:**
- `internal/lymdirs/doc.go`
- `internal/parentlock#ProcessAlive`
- `internal/state#WriteJSON`
- `internal/state#ReadJSON`
- `internal/state#UpdateJSON`
- `internal/lock#TrackedFileLockPath`

**Intent:** Add `internal/landingqueue`, the landing queue of D2: one directory per parent branch, one JSON entry per queued run, and at most one move mark.

Layout.
`lymdirs.LandingQueueDirName` names the directory under the hub's board-level `.lym`, beside `parentlock.Dir`'s.
`Dir(boardDir)` returns `<boardDir>/.lym/landing-queue`, joined from `lymdirs.DotLymDirName` and the new name.
A `Store` holds `Dir` (that directory), `Branch` (the parent branch) and an `Alive func(pid int, startTime string) bool` seam; nil judges by `parentlock.ProcessAlive`, as `parentlock` judges a holder.
Its branch directory is `<Dir>/<hex of Branch>`, keyed like `parentlock.Store.recordName`.
Each entry is `<hex of slug>.json` there and the mark is `move-mark.json`.
Every read and write goes through `internal/state` under a lock whose path `lock.TrackedFileLockPath(path, ".lock")` gives, so an unknown key a newer writer left is kept (`PATTERN-tolerant-records`).
The directory is never tracked: an absent directory, entry or mark reads as none (`PATTERN-lym-files`).

`Entry` fields, JSON-tagged:
`Slug`, `Pair` (the pair path), `RunID`, `PID`, `StartTime`, `TaskHead`, `RecordsHead` (the records head at join), `Message` (the landing message with its `Lym-Landing` trailer), `TrueMerge` (false squashes), `JoinedAt` (RFC3339 UTC, the order the leader takes entries in), `Outcome`, and `Commit`.
`Commit` is the landing commit for `landed` and the chain commit the run failed on for `failed`; it is empty otherwise.
`MoveMark` holds `Base` (the chain base) and `Members []MarkMember`, in prefix order; `MarkMember` holds `Slug`, `Commit` (its landing commit) and `Records` (the records commit its ledger landing holds).
`BackstopSeams` holds `ParentHolds func(commit string) (bool, error)`, `UpstreamLacks func(commit string) (bool, error)` and `WriteLanding func(slug, tip, records string) error`.

Behaviour.
- `Join` writes the run's entry, replacing its standing one with the new task head, owner, message and join time.
  A standing `failed` or `conflict` outcome, with its commit, is kept whatever the new task head (D2).
  A standing `landed` outcome is kept only while the new task head equals the standing one, and cleared otherwise.
- `Read` returns one entry; `Entries` returns every entry, sorted by `JoinedAt` then slug.
- `Live` reports whether the entry's owner process lives, judged by `PID` and `StartTime` through the `Alive` seam.
- `SetOutcome` rewrites one entry's outcome and commit; an `Outcome` of `""` clears both.
  An absent entry is an error naming the slug, since only a run's own entry or a batch member's is set.
- `Remove` deletes one entry; an absent one is no error.
- `Prune` removes every entry whose owner is dead and that has no outcome, and returns their slugs.
  An entry with an outcome stays until its own run reads it.
  Its caller runs it only after `Backstop`, which this card's doc comment states.
- `WriteMark`, `ReadMark` and `RemoveMark` keep the one move mark; `WriteMark` replaces a standing mark, and an absent mark reads as not found and removes as no error.
- `Backstop` reads a standing mark.
  For each member whose commit `ParentHolds` answers true and `UpstreamLacks` answers true, it calls `WriteLanding(slug, commit, records)` and sets that member's entry `landed` with the commit; a member with no entry is written to the ledger alone.
  It then removes the mark and returns how many landings it wrote.
  A mark none of whose commits the parent holds, left by a leader that died before the move, is removed with nothing written.
  A seam error returns at once with the mark kept, so the next taker of the lock runs the backstop again.

`queue_test.go` is tier0, in `package landingqueue`, and runs over `t.TempDir()` with a fake `Alive`; it adds about 1 second to one tier0 run.
`TestStore_Entries` pins, as rows of one table: an entry written and read back field for field; a rewrite of a file that carries an unknown key keeps the key; a join over `failed` and `conflict` keeps the outcome and its commit; a join with a moved task head clears `landed` and one with the same head keeps it; `Live` false for a dead owner and true for a live one; `Prune` removes a dead owner's entry with no outcome, keeps one with an outcome and keeps a live owner's; `Entries` in join order; `SetOutcome` on an absent slug errors; `Remove` of an absent entry is nil.
`TestStore_Entries` also pins that a second `WriteMark` replaces the first.
`TestStore_Backstop` pins: a mark whose two commits the parent holds and its upstream lacks writes two landings with each member's commit and records commit, sets both entries `landed` and removes the mark; a member the upstream already holds writes nothing for it; a member with no entry gets its ledger landing and no entry is created; a mark whose commits the parent does not hold writes nothing and is removed; a `WriteLanding` error keeps the mark; no mark returns 0 with no seam called.
Expected values are literals from the rows, never re-derived through the store.
`internal/landingqueue/doc.go` documents the package: the queue, its layout, its owner-liveness rule and that it is a cache.
