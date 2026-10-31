```yaml
lens: code-correctness
outcome: reported
```

Per function and branch (case run, result):

- `startReady`, hold branch (`starts.go:89`), plain released task with `StartRefusal` set, three ticks: records the refusal, spawns nothing, one notice. Correct.
- `startReady`, refusal gone next tick: record keeps fingerprint, `NextTry` zero, so `startFirstPass` runs. Correct.
- `startReady`, fingerprint changed while held: record reset, notice again. Correct.
- `startReady`, hold branch with `OwnedElsewhere`, `PendingDependencies` or a covering pause: hold runs before `readyToStart`, `heldStart` ignores all three. Not correct; see F1.
- `startReady` at `running >= MaxRuns` or `MaxRuns == 0`: a held task gets no record and no notice until a slot is free. Minor, consistent with refused starts.
- `heldStart`, existing backoff of the same fingerprint: `NextTry` and `Interval` zeroed. Correct.
- `recordRefusal`, `Notice` failure: `Noticed` not appended, retried next tick. Correct.
- `refusedStart` via `nextStartTry` as `schedule`: matches the old inline code. Correct.
- `BuildView`, held task: no `slots--`, `Refusal` copied on matching fingerprint, not `Ready`. Correct.
- `BuildView`, stale record after the refusal is gone, before the pass claims: renders `ready to start, last refused start: <done text>` for a tick. Minor, as D3 intends.
- `dobbyTaskWait`, held and ready cases. Correct.
- `DobbyTasks`, no record / done / running / undecodable. Correct per the decision record's open risk; the undecodable case warns every tick, as the run listing does.
- `doneSlugRefusal(location, slug)`: text identical when `c.location` is the prime. Correct.

Findings

F1 — LOW, design — `internal/dobbyengine/starts.go:89`
- The hold branch runs before `readyToStart` and ignores `OwnedElsewhere`, `PendingDependencies` and `covered`.
- A task owned by another machine with a done record gets a recorded refusal and a creel notice from the wrong dobby; before the diff it was skipped silently.
- Evidence: `starts.go:85-95`, `starts.go:44-52`.

F2 — LOW, design — `internal/battencli/dobbyreads.go:273-279`
- An undecodable status file logs a Warn on every tick; the run listing at `dobbyreads.go:96` does the same.

Tool calls: none.
