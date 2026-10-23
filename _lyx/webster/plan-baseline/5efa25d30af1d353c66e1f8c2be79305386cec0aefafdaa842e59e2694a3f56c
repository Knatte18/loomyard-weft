# Card 6 — records-write-lock

**Create:**
- `plan:internal/fabricengine#HoldRecordsWriteLock` -> `func HoldRecordsWriteLock(recordsWorktreePath string, wait time.Duration) (release func() error, held bool, err error)`
- `plan:internal/fabricengine#TestHoldRecordsWriteLock` -> `func TestHoldRecordsWriteLock(t *testing.T)`

**Uses:**
- `internal/fabricengine#weftWriteLockFile`
- `internal/fabricengine#RecordsLockDirPath`

**Intent:**
Export the combined records write lock every committing `Fabric.Commit` takes, so the cancel phase (D4 step 1) can hold it while it ends a run's processes, and no ended process dies inside a records commit.
`HoldRecordsWriteLock` creates the lock directory (`ensureWeftLockDirAt`) and takes `RecordsLockDirPath(recordsWorktreePath)/weftWriteLockFile` exclusively with `lock.AcquireWriteLockWithin(path, wait)`.
It returns the lock's release and held true, or held false with a nil release when the wait ran out, and an error for a failed directory creation or lock call.
`TestHoldRecordsWriteLock` is an untagged test in package `fabricengine` over a temporary directory: a free lock is held, and a second hold with a wait of a few milliseconds, never zero, since `AcquireWriteLockWithin` with a zero wait reports busy before its first try, reads not held while the first holds it, and held once the first is released.
