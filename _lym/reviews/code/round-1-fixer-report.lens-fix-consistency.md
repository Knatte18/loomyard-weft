```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set through the consistency lens, looking for stale siblings and twins left behind.

**Findings**

1. **MEDIUM** — `internal/fabricengine/doc.go:1244-1249` (edited paragraph) and `internal/fabricengine/takeoverwrite.go:108-150` (unchanged).
   - The edited doc now says "A pause therefore holds a pusher only while it holds no lock".
   - The same text says the first lock is the outermost one, and names only `Bolt.Pull`, `Bolt.PullThenCommitWritten` and `PushPairAnchored`.
   - `TakeOverRun` takes the pair's and the parent's teardown locks (`AcquireWriteLockWithin(TeardownLockPath...)`) before `takeoverBranch.land` reaches its `Admit`.
   - A covering pause therefore still holds a takeover at `land` with both teardown locks taken. That is the same defect F1 fixed for `PushPairAnchored`.
   - The summary's "Design decisions" bullet now claims a held push never holds a lock a pause could keep, and the `land` site is listed without the caveat.
   - Either gate `TakeOverRun` before its teardown locks, or scope the doc and summary sentences to say the takeover is excepted.
   - Related: the new `TakeOverRun` test probes only the push locks, so it cannot catch this.

2. **LOW** — `internal/fabricengine/coalesce.go`, the `coalescePushSides` doc (unchanged by the fix).
   - It still says the deferral is "a gate deferral that a pause triggers before the lock is taken".
   - F5 changed `pushDeferral` and `doc.go` to say it fires before each acquire, including a re-acquire after earlier steps pushed.
   - The twin sentence in `coalescePushSides` was not updated.

3. **LOW** — `internal/fabricengine/doc.go:1300-1302` (unchanged).
   - It still says a deferred push "pushes nothing more, leaves its commits local and its pending flags in place".
   - The F5 edit just below it now admits the deferral can fire after a pushed step, with flags put back for sides that have nothing left to push.
   - "Leaves its commits local" overstates that case.

**Tool calls:** none. My context already held every file I checked.
