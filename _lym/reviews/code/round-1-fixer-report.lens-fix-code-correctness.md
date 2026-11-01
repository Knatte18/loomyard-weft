```yaml
lens: fix-code-correctness
outcome: reported
```

Findings for the fix-code-correctness lens:

**LOW. Both new subtests (`resume_test.go`, "a writer not found whose outputs are missing…" and "…fail Accept…") hang instead of failing when the defect is in place.**
- Both call `engine.Resume(...)` synchronously on the test goroutine.
- If the `ender.end(lost)` branch regresses, for example a writer with missing outputs counts as finished, `Resume` queues a notice and does not end the step.
- `join` then blocks on `chair.Wait()`, and nothing in the test releases the chair. The regression shows up as a `go test` timeout panic, not as an assertion failure.
- The sibling subtests run `Resume` in a goroutine and read the result through `await(t, ended)`, which has a deadline (`resume_test.go:96-102`).
- The tests do detect the defect. The hang is only a poor failure mode, so this is LOW.
- A fix is to run `Resume` in a goroutine with `await`, as the neighbouring subtests do.

Traced, no finding:
- **Missing outputs:**
  - `allExist` is false, so `Accept` is never called and `acceptCalls` stays 0.
  - `ender.end(lost)` fires, and `join` takes the `ender.signal` branch with the chair still pending.
  - `stepEnded` is set, `sender.stop()` runs, and `markEnded` sets `EndedStep`.
  - `stopSeat(chair)` releases the fake chair as died, so `end = <-chairDone` returns.
  - `stopAdvisors` skips the lost seat because `handle == nil` (`engine.go:339`), so there is no nil dereference or wait on `done`.
  - The result has `EndedStep` true, `StrandGUID` empty (`result()` sets it only when `handle != nil`), `Outcome` empty and `AcceptErr` nil. The chair is stopped once and `sent` is empty. The assertions hold.
- **Accept error:**
  - `allExist` is true, `Accept` returns `rejected` and `accept()` records it in `acceptErr`.
  - `ender.end` fires and the rest follows the same path, with `AcceptErr` wrapping `rejected`.
  - The assertions hold.
  - If the `else` branch were broken, `EndedStep` would be false, or a notice would be queued, so the assertions would catch it, subject to the hang above.
- **Callers:** no production code changed, so no caller is affected.

Tool calls:
1. `grep -n "func stopAdvisors" … engine.go`: read it to confirm a seat with a nil handle is skipped, since my context showed only the call site.
