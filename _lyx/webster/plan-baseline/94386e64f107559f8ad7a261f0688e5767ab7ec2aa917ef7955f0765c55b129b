# Card 4 — gate-run-log

**Create:**
- `plan:internal/gateslot#RunRecord` -> `type RunRecord struct`
- `plan:internal/gateslot#RunLogDir` -> `func RunLogDir(gateDir string) string`
- `plan:internal/gateslot#WriteRunRecord` -> `func WriteRunRecord(dir string, record RunRecord, output []byte, now time.Time) (string, error)`
- `plan:internal/gateslot#TierTimeoutLine` -> `func TierTimeoutLine(tier string, timeout time.Duration) string`
- `plan:internal/gateslot#FindTierTimeout` -> `func FindTierTimeout(log string) (tier string, ok bool)`
- `plan:internal/gateslot#TierSummaryLine` -> `func TierSummaryLine(record RunRecord, logPath string) string`
- `plan:internal/gateslot#FailedTier` -> `func FailedTier(log string) (tier int, ok bool)`
- `plan:internal/gateslot#TestWriteRunRecord` -> `func TestWriteRunRecord(t *testing.T)`

**Edit:**
- `internal/gateslot#TestWatchdogLine`
- `internal/gateslot/doc.go`

**Intent:**
Keep one diagnostic record per tier run where it outlives the worktree (D11), and give a tier timeout one line a site can find in a log, as `WatchdogLine` is found today.

`RunLogDir(gateDir)` is `<gateDir>/runs`; the gate (card 8) passes `Dir(boardDir)` inside a hub and `WaitDir(worktreeRoot)` outside every hub, both already spelled through `internal/lyxdirs`, so the records sit under the board's never-tracked `.lyx` and survive the worktree's teardown.
`RunRecord` holds worktree, slug (empty without one), tier list, tier, command, run directory, scope form, slots held, wait start, start, end, wall seconds, exit code, timed-out flag, outcome (`passed` or `failed`), whether it was the guard step, and discovered, passed, failed and skipped counts, as JSON fields.
`WriteRunRecord` writes the record as `<stamp>-<tier>-<pid>.json` and the run's full output beside it as the same stem with `.log`, atomically, returns the record's path, and first removes every `.json` and `.log` in the directory last modified more than 14 days before `now`; a failed removal is logged and never returned.
The record is a domain log, read by no run to proceed (`PATTERN-durable-vs-ephemeral-state`).

`TierTimeoutLine(tier, timeout)` is the line the gate prints to stderr and into the log when a tier outlives its `timeout_min`, naming the tier and the timeout and the way forward: find the hang in the log, and when the tier is slow but healthy, rewrite the tier whole from the prime with `lyx config gate --set 'tiers.tier<N>={...}'`, carrying every key of the tier and the raised `timeout_min` (D1), since a dotted key under the tier writes an entry the loader refuses, a session that is not the prime reporting status: FAILED for the orch to run it; `FindTierTimeout` finds it in a log and returns the tier.

`TierSummaryLine(record, logPath)` is the one stderr line the gate prints after each tier run, `gate test: tier<N> <scope form> discovered <n> passed <n> failed <n> skipped <n> wall <s>s exit <code> log <path>`, and `FailedTier(log)` returns the tier number of the last such line in a log whose exit code is not 0, so a site that keeps the gate's output can name the failing tier (card 13).

`doc.go` gains a Run log section and names the timeout line beside the watchdog line.

Tests: the new `TestWriteRunRecord` covers the run-log surface, over a temp dir with an injected `now`: the record and log written with the fields read back, a 15-day-old record and log pruned while a 13-day-old one stays, the `TierSummaryLine` round trip through `FailedTier` for a log whose last failing line names `tier2` after a passing `tier0` line, and a log with no failing summary line.
`TestWatchdogLine` gains the sibling `TierTimeoutLine` round trip through `FindTierTimeout`, asserting the line names the whole-tier `--set` step, and a log without the line.

**ImpactSummary:** additive exports in gateslot with no caller yet; the existing watchdog-line test gains rows for the sibling timeout line.
