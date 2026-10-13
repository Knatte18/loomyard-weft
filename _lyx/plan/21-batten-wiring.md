# Card 21 — batten-wiring

**Edit:**
- `internal/battencli#battenCLI.wire`
- `internal/battencli#markBattenWatched` -> `func markBattenWatched(prime, taskLocation *lyxcwd.Location, scratchDir, producer string) (bool, error)`
- `internal/battencli#TestArbiterInFlightFrom`
- `internal/battencli#stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/battencli#syncWatchedMarker` -> `func syncWatchedMarker(marker string, watched bool, content []byte) (bool, error)`
- `plan:internal/battencli#TestSyncWatchedMarker` -> `func TestSyncWatchedMarker(t *testing.T)`

**Uses:**
- `plan:internal/shedrun#BattenWatchedContent`
- `plan:internal/shedrun#DriverLock`
- `plan:internal/shedrun#TacklerMarker`
- `plan:internal/shedrun#SettleRecordsFile`
- `plan:internal/loomengine#ReadTacklerMarker`
- `plan:internal/loomengine#ReadSettleRecords`
- `plan:internal/loomengine#GoVerifyRetrySpent`
- `plan:internal/loomengine#PathsOutsideRunRecords`
- `plan:internal/buildvcs#OfFile`
- `plan:internal/shedverbs#LastStepBuildChanged`
- `internal/battencli#arbiterInFlightFrom`
- `internal/battencli#runLockHeld`

**Intent:**
Wires the batten seams of cards 19 and 20 to the task worktree, and has the batten-watched marker name the prime for D4.

In `battenCLI.wire`'s `InnerRun` deps, each seam is resolved lazily through `locateTask`, as the existing ones are:
- `ChildDriverLockHeld`: `runLockHeld(shedrun.DriverLock(taskLocation, shedrun.SelfRunID))`.
- `TacklerInFlight`: `arbiterInFlightFrom` over a reader that maps `loomengine.ReadTacklerMarker(shedrun.TacklerMarker(...))` onto a `loomengine.ArbiterMarker` of the same attempt, strand and deadline, and over the same reed directory read the arbiter's seam uses, so a live row carrying the tackler's strand reads alive.
- `GoVerifyRetrySpent`: `loomengine.GoVerifyRetrySpent` over `loomengine.ReadSettleRecords(shedrun.SettleRecordsFile(loomengine.LoomFrictionDir(taskLocation)))`.
- `ChildBuildChanged`: `buildvcs.OfFile` of `os.Executable()`, returning its `Label()` with `shedverbs.LastStepBuildChanged(shedrun.StepsDir(taskLocation, shedrun.SelfRunID), onDisk)`.
  An unreadable on-disk build is returned as an error, which card 20's re-step logs and skips.
- `ChildDirtyPaths`: `fabricengine.Open(taskLocation)`'s `Status`, which lists tracked-modified, staged and untracked paths per side, each entry joined to `taskLocation.WorktreePath()`, through `loomengine.PathsOutsideRunRecords`, the read and filter the Go driver's hand-over uses (card 15).

`markBattenWatched` takes the batten scratch directory and the Run-Shed row's name, decides `watched` through `noticesReachDriversParent` as today, and calls `syncWatchedMarker` with `shedrun.BattenWatchedContent(os.Getpid(), scratchDir, producer)`.
A Go driver can then read batten's publish-retry marker through `battenshed.PublishRetrySpent` even after that batten died (D4).
`syncWatchedMarker` is the write half moved out unchanged:
- not watched removes the marker, an absent one being fine;
- watched with identical content writes nothing;
- otherwise it writes atomically.
It reports whether the marker is held.
The `markWatched` closure passes `BattenDir(location, slug)` and `battenrecipe.NameRunShed`.
Card 11's reader already accepts the three lines, so no driver misreads the longer marker.

Tests in `wire_test.go`:
- `TestArbiterInFlightFrom` gains rows reading a tackler marker through the mapped reader: live with its strand's row present, and dead without it.
- `TestSyncWatchedMarker`, a table over a temp directory:
  - watched writes the given content byte for byte (its format is `TestBattenWatchedContent`'s);
  - identical content leaves the file's modification time;
  - unwatched removes the marker, and unwatched over an absent marker succeeds.
- `stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent` in `lifecycle_integration_test.go`: its held rows expect `shedrun.BattenWatchedContent(os.Getpid(), BattenDir(h.Location, slug), battenrecipe.NameRunShed)` in place of the pid line `strconv.Itoa(os.Getpid()) + "\n"`, and the file drops its now unused `strconv` import.
The other seams are thin closures over functions with their own tests (`runLockHeld`, `ReadSettleRecords`, `OfFile`, `LastStepBuildChanged`, `PathsOutsideRunRecords`), and the tables of cards 19 and 20 pin their consumers.
`test-budget.yaml`'s `internal/battencli` row rises by one.

**ImpactSummary:** five new closures in batten's wire and a two-line longer watched marker; the re-signed `markBattenWatched` has one caller, the `markWatched` closure in the same function.
