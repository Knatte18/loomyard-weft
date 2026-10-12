# Card 1 — parentlock-store

**Create:**
- `internal/parentlock/doc.go`
- `internal/parentlock#Dir`
- `internal/parentlock#Owner`
- `internal/parentlock#Owner.SameRun`
- `internal/parentlock#Owner.SameProcess`
- `internal/parentlock#Record`
- `internal/parentlock#Ticket`
- `internal/parentlock#Entry`
- `internal/parentlock#Attempt`
- `internal/parentlock#ErrUnreadable`
- `internal/parentlock#ProcessAlive`
- `internal/parentlock#Store`
- `internal/parentlock#Store.TryAcquire`
- `internal/parentlock#Store.Read`
- `internal/parentlock#Store.Release`
- `internal/parentlock#Store.HandOff`
- `internal/parentlock#Store.List`
- `internal/parentlock#Store.ForceRelease`
- `internal/parentlock#Store.ReleaseAll`
- `internal/parentlock/store_test.go`
- `internal/parentlock/seam_enforcement_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`
- `pattern/PATTERN-told-geometry.md`
- `docs/overview.md`

**Intent:**
Add `internal/parentlock`, the told-geometry engine that owns the parent lock (D4): one record per parent branch, valid exactly while its owner process lives, so a host that sleeps keeps it and a crashed owner frees it at once.

- `Dir(boardDir)` returns `<boardDir>/.lyx/parent-lock`, spelling the ephemeral directory name only through `lyxdirs`, as `gateslot.Dir` does.
  Every other path is a join under the `Store.Dir` the caller tells; the package never imports `lyxcwd`.
- `Owner` names the owner run (`Slug`, `Pair`, `RunID`) and the owner process (`PID`, `StartTime`).
  `SameRun` compares the three run fields; `SameProcess` compares the run fields and the pid and start time.
- `Record` is `Branch`, `Owner` and `AcquiredAt`, stored as JSON at `<Dir>/<hex(branch)>.json`.
  The branch is hex-encoded into the file name, so a name with `/`, `..` or a case clash on Windows cannot escape or collide in the directory.
  The first write creates `Dir`.
- `Store` carries `Dir`, `Alive func(pid int, startTime string) bool` (nil selects `ProcessAlive`, which is `proc.IsAlive` plus an equal `proc.StartTime`) and `Now func() time.Time` (nil selects `time.Now`).
  `Now` stamps `AcquiredAt` and ticket times only; validity never reads a clock.
- Every read-modify-write runs under `lock.AcquireWriteLock` on `<Dir>/<hex(branch)>.lock`, so two acquirers can never both win.
- `TryAcquire(branch, owner)` is one attempt.
  A free record, a record whose owner process is not alive (dead pid, or the pid now carries another start time), and a record naming the same run (`SameRun`) are each taken: the record is rewritten with `owner`, so a new loop of the same run, or the loop taking a record back from batten, rewrites the owner process.
  Waiters queue through tickets at `<Dir>/<hex(branch)>.tickets/<pid>-<start>.json`, each carrying the waiter's `Owner` and its queue time: a free or stale record goes to the oldest live ticket, so an acquirer whose own ticket is not the oldest live one writes or keeps its ticket and is not granted.
  A dead waiter's ticket is skipped and removed in the same attempt.
  A granted acquirer's own ticket is removed.
  `Attempt` reports `Granted` and the `Holder` record the attempt found when not granted.
  An unreadable or undecodable record returns an error wrapping `ErrUnreadable` naming the record path, and changes nothing.
- `Read(branch)` returns the record, found false when none exists.
- `Release(branch, owner)` is a compare-and-release: it removes the record only when its owner `SameProcess` matches `owner`, and reports whether it removed one; a record owned by another process of the same run (batten, or a newer loop) is left alone.
- `HandOff(branch, from, to)` rewrites the owner to `to` only when the record's owner `SameProcess`-matches `from`, and reports whether it did.
- `List()` returns one `Entry` per record in `Dir`: the `Record`, `Alive` for its owner and the live `Waiters` tickets oldest first; an unreadable record lists with its error text instead of failing the whole list.
- `ForceRelease(branch)` removes the record whatever its owner, reporting false when there was none; `ReleaseAll()` force-releases every record and returns the released branch names, the function `lyx pause` (note `lyx-halt`) will call (D11).
  Bound: a force release removes the record only and never stops the owning run or touches its tickets, and the released run reacquires at its next lock-taking row, where the moved-parent check runs.

`store_test.go` pins the acceptance criteria at the package surface with a fake `Alive`: two goroutines racing `TryAcquire` on a free lock produce exactly one grant; three waiters are granted in ticket order across releases, skipping one whose `Alive` turns false; a record whose owner pid is dead, and one whose pid reports another start time, is taken over at once; a branch `feature/x/../y` maps to a file inside `Dir` and the first acquire creates `Dir`; a `Now` that jumps a day forward during the hold leaves the record held against another run; a same-run acquire rewrites the owner process; `Release` and `HandOff` leave a record owned by another process alone; an undecodable record returns `ErrUnreadable`; `List` reports liveness and waiters; `ReleaseAll` returns every branch.
`seam_enforcement_test.go` pins the import allowlist with `scankit.AssertImportAllowlistNoStale`, as every told-geometry package does: the standard library, `lock`, `proc`, `lyxdirs` and `logger`.

`doc.go` documents the record, the ticket queue, liveness by pid and start time, the release and hand-off compare rule, the force releases, and that no clock decides validity.
`pattern/PATTERN-told-geometry.md` lists `parentlock` among its bound packages, `docs/overview.md`'s module table gains its row, and the test budget gains the package's row.

**ImpactSummary:** a new leaf package with no callers yet; the budget, the told-geometry background and the overview table each gain one line.

**Commit:** 1: parentlock: per-branch parent lock record with ticket queue and pid liveness
