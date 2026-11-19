# Card 6 — live-gate

**Create:**
- `plan:internal/hardenshed#LiveDeps` -> `type LiveDeps struct`
- `plan:internal/hardenshed#LiveRun` -> `type LiveRun struct`
- `plan:internal/hardenshed#LiveReport` -> `type LiveReport struct`
- `plan:internal/hardenshed#NewLiveGate` -> `func NewLiveGate(deps LiveDeps) shuttleengine.Gate`
- `plan:internal/hardenshed#ReadLiveReport` -> `func ReadLiveReport(runDir string, round int) (LiveReport, bool, error)`
- `plan:internal/hardenshed#TestLiveGate` -> `func TestLiveGate(t *testing.T)`

**Extends:**
- `internal/hardenshed/doc.go`

**Uses:**
- `internal/gateslot#ParseStartedLines`

**Intent:**
The `live` gate, Go's independent verification of each fix round (D4).

`LiveDeps` carries the run directory, the worktree root the `packages` resolve against, a `Round func() (int, error)` naming the round being fixed, `packages`, `llm_driving`, `concurrency`, and `Run func(key string, copies int, packages []string) (LiveRun, error)`.
Card 14 wires `Run` to `lym gate test --wait --gate <key> [--copies <n>] <packages...>`, spawned through `selfexec.LymBinary` as a top-level call that takes its own slots, never nested in a held slot.
`LiveRun` holds whether the call passed, its output, and `Refused`, the refusal text when the call refused before running any tier, empty otherwise.
The gate names keys only, never a tier.

Each call of the gate:
1. Runs `harden-live` once with copies 0.
   A refused call, `Refused` set, is returned as an error wrapping the refusal text, whose own way forward it keeps, and the gate writes no report, so the fixer is never re-prompted over a hub config fault.
2. When `llm_driving` is false and `concurrency` is above 1, runs `harden-live-copies` with `concurrency` copies; the call itself clamps the copies to the pool.
   A refused copies call is recorded in the live report as refused, with its refusal text, never as a failed run, and leaves the mark unchanged.
3. Reads each run's started counts through `gateslot.ParseStartedLines`; a run whose output carries no started line counts as having started 0 test binaries.
4. Writes the round's live report, `harden-live-<round>.json` in the run directory: per run, passed, the started count, the failing lines of its output and, for the copies run, the clamp line when one printed or the refusal text when the call was refused; the packages among `packages` whose test files, read under the worktree root, hold no `tmuxkit.Main(` call, named as outside the stray-state cover; and the mark.
   The mark is red when the `harden-live` run failed or started no test binary, and green otherwise: the copies run never changes it, and a copies run that started none is recorded beside its failures.
   A red report lists the `harden-live` run's failing lines as deferred findings.
5. Fails the gate when the mark is red, its findings naming the report and the failing lines; passes otherwise.

A `Run` error, a spawn or wait failure, is returned wrapped with "way forward: check the key with lym config gate, fix it from the prime, then run lym loom resume", and writes no report.
The gate never runs tier 3: Preflight refuses a live key naming it (card 5), and the gate passes no tier.
Card 11 builds the gate's entry with `live_attempts` attempts and `pass_on_cap`, so once the attempts run out the row ends with the last written report red and its failures deferred, and the run goes on.
`ReadLiveReport` is the report's sole reader; card 11's facts and card 10's auto-accept bound read it.

`TestLiveGate` is tier0 with a fake `Run` over a temporary worktree root:
- a green `harden-live` passes with a green report;
- a red one fails with a red report listing its failing lines as deferred;
- a `harden-live` run whose started line names 0 fails red, and so does a passing one whose output carries no started line;
- a red copies run passes and is recorded with the mark green;
- a copies run that started none is recorded and the mark stays green;
- a copies run's clamp line is recorded in the report;
- with `llm_driving` true or `concurrency` 1 no copies call is made;
- a package under the worktree root without `tmuxkit.Main` is named outside the cover, and one with it is not;
- a `Run` error is returned with its way forward and no report is written;
- a refused `harden-live` call is returned as an error carrying its refusal text and no report is written;
- a refused copies call is recorded in the report as refused, not failed, and the mark stays green.

**ImpactSummary:** New gate and report in hardenshed; nothing existing reads or calls them until card 11.
