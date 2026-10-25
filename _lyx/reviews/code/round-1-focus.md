---
round: 1
focus:
  - "internal/dobbyengine/passes.go drivePasses: does filtering done runs before recordOwed keep a done run's Runs entry out of state.json in every path, including a tick where a dobby start owed every run a full pass, and does a done run with a pass in flight keep its slot and entry until the exit is read?"
  - "internal/dobbyengine/starts.go runsInFlight and its callers: is every count and pass-planning path fed from the filtered d.runs, and does anything else (busy flag, owned-elsewhere handling, first-pass bookkeeping) still read the unfiltered list?"
  - "internal/dobbyengine/starts_test.go new case: does it pin the acceptance criteria (done records beside a released ready task with max_runs at or below the done count starts the task; no state entry for done runs; in-flight done run keeps its slot), on the injected clock as a tier0 test, without duplicating existing coverage?"
  - "internal/dobbyengine/doc.go: do the passes and starts bullets match the new behavior, and does the diff leave the board tidy landing count and beamcli listing untouched?"
---

Seed focus for round 1 over the `dobbyengine` diff; the answer key is the board-entry snapshot and the decision record.
