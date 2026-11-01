```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: for each finding fixed as proposed, I tried to build a case where the reviewer's proposed change is itself wrong.

**MEDIUM — F3's wall-time definition fails for any call that runs a build step**
- **Changed text:** D5, "A call's wall time is the span from the earliest `WaitStart` to the latest `End` over the records sharing its call id". The D5 acceptance criterion says the same.
- **Case:** a `--gate` call whose first opted-in tier builds before it runs. The build step's record is written at `internal/gatecli/build.go:65` with `Start`, `End`, `WallSec` and `Build: true`, but no `WaitStart`. That record's `WaitStart` is the zero time. It shares the call id, so the earliest `WaitStart` is year 1. The call's wall time comes out as about two thousand years, so it pushes the median past cap times `keep_warm_max_reruns`. The front then always takes the 55m long cap for that key.
- **Evidence:** `internal/gatecli/report.go:147` already guards `if !record.WaitStart.IsZero()` for this reason. The definition needs the same guard: leave out zero `WaitStart` values, or fall back to `Start`.

**LOW — F2's "one write that moves it earlier" turns a supersede into a false failure for another front still waiting**
- **Changed text:** D3, "the supersede expiry (D1) is the one write that moves it earlier".
- **Case:** front A, from one fork, is waiting with `AwaitAny` on run R's exit file. Front B, from another fork of the same session, has the same words. B's session changed a file, so the tree or dirty set differs. B expires R's lease and starts R2. R reads the passed lease, cancels, and writes exit 1 with "no caller came back" (D3). A sees that exit file, replays exit 1 and returns a failure no test produced. This breaks D5's "never a verdict" for that path.
- **Note:** the path existed before this round. The fix made it explicit without stating the cost or giving A a way out, for example writing the superseded run's exit as 75 `done: false` so A runs the same command again and joins R2.

**LOW — F1's 128-plus-signal exit can be replayed to a front whose call nobody signalled**
- **Changed text:** D3, "A terminating signal to the runner cancels its call the same way, and its exit file records 128 plus the signal number".
- **Case:** an operator, or `lym tender`, ends the registered gate-role runner to free slots while its lease still holds. The agent's next same-words call joins the run: the lease holds, the result is not delivered, the tree matches, and the exit file is present. It replays exit 143. D9's stencils read every non-75 code as a failure, so the agent records a failed gate although its own front got no signal and no test ran to an outcome.
- **Note:** D1's join rule could instead treat a run ended by a signal like a superseded one and start fresh.

The other proposals (F4, F5, F6, F7, F8, F9, F10, F11) gave me no concrete breaking case.

Tool calls:
- One grep of `WaitStart`, `Build`, `Carried` and `reportBlocked` across `internal/gatecli/*.go`, to see which run records leave `WaitStart` zero. My context held only `run.go`'s `report`, not `build.go` or `report.go`.
