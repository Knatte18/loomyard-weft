```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each review proposal the fixer carried out as proposed, using the fix-proposal lens.

Proposals I checked: F1, F2, F3, F5 (ruling), F6, F7, F9, F10, F11, F12, F13, F14, F15 (exemption branch). F4 and F8 were fixed differently, so they fall outside this lens.

**[BLOCKING] F6 (card 8, `readCancelMarker` bullet): the shared-lock try now runs on every read, and it fails wherever the scratch root is missing.**
- Changed text: "So `readCancelMarker` tries the shared lock on every read, not only once an intent is seen."
- Concrete case: a fresh machine, a prime that has never run a cancel, or a prime whose `.lym` was removed (D3's own criterion). Its `ScratchRoot` does not exist, and `CancelMarkerLock` is `filepath.Join(ScratchRoot(l), …+".cancel.lock")` (`internal/shedrun/paths.go:463-464`).
- `lock.TryAcquireReadLock` calls `flock.New(lockPath).TryRLock()` and never creates the parent directory (`internal/lock/lock.go:94-107`). Only `takeCancelMarker` runs `os.MkdirAll(shedrun.ScratchRoot(location), …)` (`cancelmarker.go:133`).
- So `readCancelMarker` returns an error for every slug. Through it, the start guard, the pre-drive check and `DobbyRuns` fail every run. That contradicts D3: "An absent cache reads as empty and refuses nothing."
- Today the try runs only after the marker file stats present, so the directory always exists at that point.
- The fix also leaves a lock file behind for every slug that is read.
- Remedy: an absent lock file or scratch directory reads as free, with no cancel under way. Add a `TestCancelMarker` row for it.

No other proposal breaks under a concrete case:
- **F12:** the board-`done` slug with no batten record does reach the verdict. Card 7 asks the verdict right after the reached-parent check, and `refuseNothingLeft` runs later (`cancel.go:310-328`).
- **F1:** the started set {Run-Shed, Land, Worktree-Teardown} matches batten's rows. The pawl rows belong to the child run, not to the batten record.
- **F7, F11, F15:** each case I built either sets the relevant input aside first or is a behaviour that exists today and that the card's bound already states.

Tool calls:
- grep of battencli for the pawl recipe and Beam-Await: to check whether a beam run's batten record carries pawl rows (relevant to F1); my context lacked it.
- grep of `cancel.go`'s refusal order, `CancelMarkerLock` and `TryAcquireReadLock`: to test the F12 and F6 cases; my context lacked those spans.
- read of `cancel.go:273-345` and `lock.go:94-107`: the `endRunPhase` order and the lock's create behaviour, which my context lacked.
