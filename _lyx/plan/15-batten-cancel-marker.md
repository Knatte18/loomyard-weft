# Card 15 — batten-cancel-marker

**Create:**
- `plan:internal/battencli#cancelMarker` -> `type cancelMarker struct`
- `plan:internal/battencli#cancelMarkerReading` -> `type cancelMarkerReading struct`
- `plan:internal/battencli#readCancelMarker` -> `func readCancelMarker(location *lyxcwd.Location, slug string) (cancelMarkerReading, error)`
- `plan:internal/battencli#heldCancelMarker` -> `type heldCancelMarker struct`
- `plan:internal/battencli#takeCancelMarker` -> `func takeCancelMarker(location *lyxcwd.Location, slug string, wait time.Duration) (heldCancelMarker, error)`
- `plan:internal/battencli#heldCancelMarker.release` -> `func (h heldCancelMarker) release(remove bool) error`
- `plan:internal/battencli#cancelMarkerRefusal` -> `func cancelMarkerRefusal(slug string, reading cancelMarkerReading) error`
- `plan:internal/battencli#judgeCancelMarker` -> `func judgeCancelMarker(presentBefore, lockFree, presentUnderLock bool) (present, writerAlive bool)`
- `plan:internal/battencli#TestCancelMarker` -> `func TestCancelMarker(t *testing.T)`

**Uses:**
- `plan:internal/shedrun#CancelMarker`
- `plan:internal/shedrun#CancelMarkerLock`

**Intent:**
Implement D3's cancel marker in its own file, `cancelmarker.go`: a record that a cancel of the slug is under way, whose writer's liveness is a lock it holds, never its pid.

`cancelMarker` is the JSON record at `shedrun.CancelMarker`: `pid`, `start_time` (`proc.StartTime`, empty on Windows) and `started_at`, written through `internal/state` (PATTERN-tolerant-records).
`cancelMarkerReading` carries `Present`, `WriterAlive` and the decoded `Marker`.

`readCancelMarker` stats the marker; absent reads not present.
Present, it try-acquires `shedrun.CancelMarkerLock` shared (`lock.TryAcquireReadLock`): not acquired reads present with the writer alive; acquired, it stats the marker again while holding the shared lock, reading not present when the writer removed it meanwhile and present with the writer gone otherwise, then releases.
The decision over those three readings is `judgeCancelMarker`: absent before reads not present; present with the lock held elsewhere reads present with the writer alive; present with the lock free reads present with the writer gone when the marker is still there under the shared lock, and not present when it is not.
A marker that does not decode reads present with its fields empty, so a torn write never hides a cancel.

`takeCancelMarker` creates the lock's directory and takes the lock exclusively with `lock.AcquireWriteLockWithin(path, wait)`, so a reader's shared probe that holds it briefly is waited out; not taken within the wait refuses as a live cancel (`cancelMarkerRefusal` with the writer alive).
Holding it, it records whether a marker was already there (`fresh` false for a taken-over marker of a dead writer) and writes the marker with this process's pid, start time and the time now.
`heldCancelMarker.release(remove)` removes the marker first when `remove` is true, then releases the lock, so the marker never outlives its lock while being removed.

`cancelMarkerRefusal` words the refusal a driving verb or a second cancel gets, each ending in a trailing `way forward:` clause (PATTERN-refusal-way-forward), since it reaches callers through shed's step verb (cards 7 and 17): with the writer alive, `a cancel of <slug> is under way (pid <pid>, since <time>); way forward: wait for the cancel (pid <pid>) to end, seeing it with "ps -p <pid>" or in Task Manager on Windows, then run the verb again`; with the writer gone, `a cancel of <slug> was interrupted; way forward: run "lyx batten cancel <slug>" from the prime to finish it`.

`TestCancelMarker` is an untagged test over a temporary location: no marker reads absent; a taken marker reads present with its writer alive (the exclusive lock held on another file handle in this process) and its refusal ends with the `way forward:` clause naming the wait, the pid and `ps -p <pid>`; a second `takeCancelMarker` with a zero wait refuses; after the holder's lock is released with `remove` false the marker reads present with its writer gone, whatever pid it records, and the refusal ends with the `way forward:` clause naming `lyx batten cancel <slug>`; taking it over reports `fresh` false; `release(true)` leaves no marker; an undecodable marker reads present.
Its `judgeCancelMarker` rows pin each reading, the writer removing the marker between the first stat and the shared lock included, which the file-level rows cannot time.
