# Card 3 — sweep-retry-budget

**Create:**
- `plan:internal/lyxtemp#removeRetryBudget` -> `const removeRetryBudget = time.Second`
- `plan:internal/lyxtemp#isRemovalSymptom` -> `func isRemovalSymptom(err error, path string) bool`
- `plan:internal/lyxtemp#retryClock` -> `type retryClock struct`
- `plan:internal/lyxtemp#retryClock.now` -> `func (c *retryClock) now() time.Time`
- `plan:internal/lyxtemp#retryClock.sleep` -> `func (c *retryClock) sleep(d time.Duration)`

**Delete:**
- `internal/lyxtemp#removeAttempts`

**Edit:**
- `internal/lyxtemp#Root`
- `internal/lyxtemp#openRoot`
- `internal/lyxtemp#Root.Sweep`
- `internal/lyxtemp#removeRunDirectory` -> `func removeRunDirectory(r *Root, path string, deadline *time.Time) error`
- `internal/lyxtemp#removeRetryDelay`
- `internal/lyxtemp/doc.go`
- `internal/lyxtemp#TestSweep`

**Intent:**
The sweep's removal retries only the lock-file recreation symptom, under one time budget per `Root.Sweep` call instead of a fixed three attempts (D7), so many concurrent sweepers no longer fail `lyx cleanup`'s temp segment.

`Root` gains three unexported fields beside the creation-grace `now`: `remove func(string) error`, `retryNow func() time.Time` and `sleep func(time.Duration)`.
`openRoot` sets them to `os.RemoveAll`, `time.Now` and `time.Sleep`, so `ProcessRoot` and every test built on `openRoot` get the real behavior unchanged.

`isRemovalSymptom`, in `sweep.go`, reports whether `err` is an `*fs.PathError` whose `Path` cleans to `path` itself and whose `Err` is `fs.ErrExist`: the directory-not-empty failure of removing the directory itself.
Both `syscall.ENOTEMPTY` and `syscall.EEXIST`, the two errors POSIX allows `rmdir` of a non-empty directory to give, and Windows' `ERROR_DIR_NOT_EMPTY` satisfy `fs.ErrExist`, so the predicate needs no build-tagged file; its Windows behavior stays unverified, as the record's open risk says.
A not-empty error on a path inside the directory is no symptom.

`removeRunDirectory` takes the `Root` whose seam it calls and the sweep's shared `deadline`, zero until the sweep's first symptom; it stays a function, since a method would rename its glyph.
It calls `r.remove(path)`; success, or a path that `os.Lstat` then finds gone, is removal.
Any other failure that is not the symptom returns after that one attempt.
On a symptom it sets a zero `*deadline` to `r.retryNow()` plus `removeRetryBudget`, and while `r.retryNow()` is before the deadline it calls `r.sleep(removeRetryDelay)` and tries again; once the deadline has passed, the symptom error returns.
`Root.Sweep` declares one `deadline` per call and passes it to every removal, so a second directory hitting the symptom after the budget ran out gets one attempt and no fresh budget.
`removeAttempts` goes; `removeRetryDelay`'s comment says it is the pause between symptom retries.

What the retry can now let through, and its bound: a sweep waits at most `removeRetryBudget` plus one `removeRetryDelay` and one removal in total across all its directories, reached only while something keeps re-adding entries to a dead directory, such as a concurrent sweeper's lock probe or a process outliving the directory's dead owner, and paid by every `newRun` for as long as that lasts; a persistent failure of another kind costs one attempt per dead directory.
The seam is unexported and uses only the standard library, inside the package's leaf import set that `TestLeafInvariant_AllowlistOnly` enforces.

`doc.go` gains, after the sweep's candidate sentences: a removal that fails because the directory is not empty, the symptom of a concurrent sweeper's lock probe recreating the lock file, is retried until one budget shared by the whole sweep runs out, then reported as a failure; any other removal error is reported after one attempt; and the sentence above as its bound.

Tests, as subtests of `TestSweep` through `Root.Sweep(true)` with no real sleeping:
- `retryClock`, in `sweep_test.go`, starts at `fixedClock()`; `sleep` advances it by the duration, so a row sets `root.retryNow` and `root.sleep` to its methods.
  Each row sets `root.remove` to a closure over a per-path attempt counter, returning `&fs.PathError{Op: "unlinkat", Path: path, Err: syscall.ENOTEMPTY}` for the symptom.
- "symptom past three attempts then removed": the symptom five times, then `os.RemoveAll`; no failure, the directory is a candidate and gone, and six attempts ran.
- "symptom past the budget fails and shares it": two dead directories and an always-symptom remove; both are reported as failures whose error `errors.Is` `syscall.ENOTEMPTY`, both stay in place, the first took exactly `int(removeRetryBudget/removeRetryDelay) + 1` attempts, which pins the budget's length, and the second exactly one.
- "another error is reported after one attempt": `syscall.EACCES` on the path itself; one failure, one attempt, the directory in place.
- "not-empty on a path inside is another error": `syscall.ENOTEMPTY` with `Path` inside the directory; one attempt and a failure.
- "a path gone between attempts counts as removed": the closure calls `os.RemoveAll`, then returns the symptom; no failure and the directory is a candidate.
The existing concurrent subtest keeps the real defaults and still pins two real sweepers both succeeding.

**ImpactSummary:** Confined to `internal/lyxtemp`: `removeRunDirectory`'s only caller is `Root.Sweep`, `openRoot`'s callers keep their signature, and the sweep's public surface and report shape are unchanged.

**Commit:** 3: lyxtemp: retry a sweep removal's not-empty symptom under a per-sweep budget
