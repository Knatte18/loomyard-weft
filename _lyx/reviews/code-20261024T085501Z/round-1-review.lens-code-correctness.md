```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens, covering `ReadAgentActivity` and `noteWaiterlessRun` in `internal/shuttleengine/activity.go`.

**Per branch, case run, verdict**

| Branch | Case run | Verdict |
|---|---|---|
| Record unreadable (`activity.go:79-83`) | A waiterless episode is open, then one torn read, then a readable record that is still waiterless. The torn read leaves `waiterlessRuns` alone, so the next waiterless read is a Debug line with a continued count. | Correct. |
| Record not found (`:85`) | A run dir that logged a Warn loses its `run.json`, or the dir is removed. The `waiterlessRuns` key is never recovered, so there is no Info and the map entry stays for the life of the process. | See F1 (LOW). |
| `!runIsLive`, running, pid>0 (dead) | Read 1 gives Warn with `runDir`, `strand` and `pid`. Reads 2 and 3 give Debug with `failed_ticks`. | Correct. The skip is kept. |
| `!runIsLive`, running, pid 0 | Goes to `Recover` with no episode open, so it logs nothing, as the acceptance criteria require. | Correct. |
| `!runIsLive`, outcome `done` or other | With no episode open, nothing is logged. With one open, a single Info is logged and the key is deleted. | Correct. |
| `!runIsLive`, running, pid alive | Not reachable here, because `runIsLive` is true for it. | Not applicable. |
| Live (`:92`) | `Recover` runs on every poll of every live run: a mutex lock and a map lookup, and it logs only when an episode is open. After a waiterless to live transition the next read gives exactly one Info, the key is deleted, and a later waiterless read warns again. | Correct. |
| `engineOfRun` fails after `Recover` | The waiterless Info is logged first, then the unknown-engine episode logs as before. The two sets have separate keys, so neither interferes with the other. | Correct. |
| Existing episodes (`unreadableFiles`) | The new code never touches `unreadableFiles`. The decision's reason for a separate set is sound: the unreadable-record `Recover` at `:84` would otherwise end the new episode on every readable read. | Correct. |
| Concurrency | `EpisodeSet` is mutex-guarded, and `noteWaiterlessRun` holds no state of its own. | Correct. |
| Return value and liveness | The skipped-run set is unchanged and the record shape is unchanged. The `continue` after `noteWaiterlessRun` keeps the old behaviour. | Correct. |

**Findings**

- **F1, LOW, design.** `activity.go:85-87` and `:110`. The `!found` branch does not recover `waiterlessRuns`.
  - A run dir whose record vanished while its episode was open (for example, the dir was pruned) leaves a stale episode key and never logs the pairing Info.
  - Evidence: the `!found` branch `continue`s before any `Recover` call, and `noteWaiterlessRun` is only reached for found records.
  - The effect is a missing Info plus one leaked map entry per process lifetime, so the severity is low.
  - A fix is to call `waiterlessRuns.Recover` in the `!found` branch, or accept and document the gap.

- **F2, NIT, consistency.** `activity.go:113`, used at `:92` and `:124`. The recovery message says "no longer reads running with its waiting process gone".
  - On the live-again path the record still reads running and its waiter is alive, so "no longer reads running" is inaccurate there.
  - The decision record describes both triggers ("reads live again or no longer reads `running`").
  - A neutral wording, such as "run no longer waiterless", would fit both.

**Test setup, not verified**
- The new test marks pid 4343 as dead via `isAlive` in `activity_test.go:82`, which is true only if the package-level `livePID` differs from 4343. I did not confirm that value: the only `livePID = 4343` I saw is a function-local constant in `agentwait_test.go:35`. A wrong value would make the test fail at the gate, not produce a wrong result.

**Tool calls (2)**
1. `grep` of `livePID` in the shuttleengine tests, to check which pid the `isAlive` stub treats as alive. Context lacked it.
2. `grep` of the `isAlive` definition, to confirm it is a stubbable package var (`waitmark.go:32`). Context lacked it.
