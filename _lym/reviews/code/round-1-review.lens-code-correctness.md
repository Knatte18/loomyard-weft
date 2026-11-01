```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each function or branch the diff changes with a concrete input.

**Per-branch list**
- `Seed`, fresh path: writes empty `CurrentProducer`, state running, empty non-nil `History`. Correct.
- `Seed`, existing or undecodable file: unchanged, `ErrSeedExists` path. Correct.
- `RowAt(producers, "")` with three producers returns `producers[0].Name`. Correct.
- `RowAt(nil, "")` returns "". `stepLocked` then fails with the "names no producer" error and nothing is written. Correct.
- `RowAt(producers, "X")` returns "X", so the named-row-not-found error is unchanged. Correct.
- `stepLocked` with an empty seed and an ungated first row: the in-memory `st.CurrentProducer` is resolved, so every persist names the row. Correct.
- `stepLocked`, pause_before on the first row: compares the resolved name and persists paused at that name. Correct, and the new test covers it.
- `stepLocked`, pause_requested or a cancelled context before the first row: persists the resolved name. Correct.
- `stepLocked` with state done and an empty producer: returns `Next: ""`. This needs a seed that is already done, which is not reachable. Correct.
- `stepLocked`, first row gated, gate holds, empty seed: **incorrect**, see F1.
- `stepLocked`, first row gated, gate does not hold: `persistRouted(def.OnDone...)` writes a named row, and the later re-read gets a named `CurrentProducer`. Correct.
- `Routing.ProgressAt("")`: resolves to the first row and gives Step 1. Correct. With no producers it returns "" unchanged, and the callers `progressOf` and `human.go` guard `len(Producers) > 0`.
- `Goto` and `Relaunch` on a fresh seed: both refuse a running state, so an empty producer never reaches them. `gotoAdmitted` already falls back to the first row. Correct.
- `RenameProducers` and `namesRetiredRow` with an empty `CurrentProducer`: `renamed("")` returns "". Correct.
- `DetectCrashResume`: a fresh seed has history length 0 and is excluded. Correct.
- `driver.checkBlockedStart` and `resumeOpenAttempt`: only act on non-running states, which always carry a name after the first persist. Correct.

**Findings**

F1 — MEDIUM, design — `internal/shedengine/run.go:240` and `:278-289` (with `internal/shedengine/entry.go:46-50`, `:88-97`)
- **Summary:** when the first row has an `EntryGate` that holds, the resolved `CurrentProducer` is lost, and a halt on that row persists an empty `current_producer`.
- **Trace of the case I ran:**
  1. A fresh seed has `current_producer` "" and the first row has an `EntryGate`.
  2. `stepLocked` resolves the producer in memory only (`run.go:240`).
  3. `gated` is true, and `enter` calls `markEntered`, which updates only `EnteredRow` on disk (`entry.go:88-97`).
  4. The `if gated` re-read (`run.go:278-289`) overwrites `st` with the disk copy, so `st.CurrentProducer` is "" again.
  5. If the producer then returns stuck, fails, awaits or blocks, `persist(st.CurrentProducer, ...)` writes `current_producer: ""` with a halted state and history (`run.go:400`, `:414`, `:430`, `:453`, `:498`, and `reroute.go:47`, `:68`).
  6. `StepResult.Next` is also "" (e.g. `run.go:417`).
- **Consequences:** a halted run records no row. The stop key, the arbiter's `InterruptPolicy("")`, `StartCheck("")`, the tackler's `gotoRows` and `Activity.Now` all see an empty producer. The next step reads "" as the first row, which is only right while the halt row is the first one.
- **Fix:** re-apply `st.CurrentProducer = RowAt(s.Producers, st.CurrentProducer)` after the re-read in the `if gated` block, or have `markEntered` also write the resolved row. Add a test for a gated first row on an empty seed that halts, asserting the persisted `CurrentProducer`.
- **Reach:** this applies only if some recipe's first row is gated. The fork did not check which recipes gate their first row.

**Tool calls**
1. `sed -n 1,110p internal/shedengine/entry.go`: needed `enter` and `markEntered` to see what the gated re-read returns; the manifest lacked entry.go.
