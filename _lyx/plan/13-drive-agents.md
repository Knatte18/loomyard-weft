# Card 13 — drive-agents

**Edit:**
- `plan:internal/loomcli#goDriverDeps`
- `plan:internal/loomcli#goDriver.run`
- `plan:internal/loomcli#goDriver.facts`
- `plan:internal/loomcli#TestGoDriver_Settle`

**Create:**
- `plan:internal/loomcli#goDriver.tackle` -> `func (d *goDriver) tackle(ctx context.Context, stop driveStop) (next driveStop, exit bool, err error)`
- `plan:internal/loomcli#goDriver.arbitrate` -> `func (d *goDriver) arbitrate(ctx context.Context, stop driveStop, caseBriefPath string) (next driveStop, exit bool, err error)`
- `plan:internal/loomcli#goDriver.resumeOpenAttempt` -> `func (d *goDriver) resumeOpenAttempt(ctx context.Context) (stop driveStop, found bool, err error)`

**Uses:**
- `plan:internal/loomcli#errTacklerUnavailable`
- `plan:internal/loomcli#errTacklerResumeReturned`
- `plan:internal/loomcli#goDriver.recheck`
- `plan:internal/loomcli#goDriver.holdPause`
- `internal/loomengine#WorktreeSnapshot.Equal`

**Intent:**
D6, D7 and D12 inside the loop: the tackler at the stops no rule settles, the arbiter at the three stops D6 lists, and a restart that reuses an open attempt instead of spawning a second agent.
The code lands in `internal/loomcli/drive.go`.

`goDriverDeps` gains six seams:
- `Tackler(stop driveStop, reportPath string) (loomengine.TacklerEnvelope, error)`, which builds `tacklerDeps` (its `Closed` over the settle records) and calls `newTacklerCall(...).settle()`;
- `ConsumeTackler(attemptDir string) error`, wrapping `consumeTacklerAttempt`;
- `Arbiter(envelopePath, reportPath, caseBriefPath string) (loomengine.ArbiterEnvelope, error)`, which builds `arbiterDeps` and calls `newArbiterCall(...).settle()`, the same path `lyx loom arbiter` runs, so eligibility, `arbiter_max_spawns`, attempts, the applied mark and deadlines are shared (D6);
- `Snapshot() (loomengine.WorktreeSnapshot, error)`;
- `OpenAttempts(key loomengine.ArbiterStopKey) (tackler, arbiter string, tacklerApplied bool, err error)`, naming the newest tackler and arbiter attempt directories recorded for the key;
- `AttemptSnapshot(tacklerDir string) (loomengine.WorktreeSnapshot, error)`, the spawn snapshot of a tackler attempt.
`facts` sets `ResumeReturned` from `tacklerApplied`.
In `run`, the tackler action calls `tackle`, the arbiter action and a returned tackler `resume` call `arbitrate`, and the loop goes on with the stop they return.

`tackle`:
1. Holds any pause first and writes the stop report so far at the attempt's report path for the tackler to read.
2. Calls `Tackler`. Once it returns, a `spawn` settle record with settler `tackler` is written only when the call started a fresh attempt, that is, when the returned attempt number is not among the attempts `OpenAttempts` or `ListTacklerAttempts` listed before the call; a refusal, a reused verdict or an attached attempt writes none. An `errTacklerUnavailable` hands back, the reason in the report; `errTacklerResumeReturned` goes to `arbitrate`.
3. With a verdict in hand, it runs `recheck` (D12) before anything else:
   - a changed stop key is a moved run: the verdict is discarded, the stop report gets a line noting the change, and the fresh state is acted on as a new stop, never as `tampered`, since the records cannot say which actor moved the run;
   - with the key unchanged, an attempt snapshot that is not `Equal` to `Snapshot()` reads as `hand-back` with cause `tampered`.
4. Acts on the verdict:
   - `resume` calls `ConsumeTackler`, and a refusal is acted on as a new stop through `recheck`; once consumed, it writes a `restep` record with settler `tackler` and steps;
   - `hand-back` hands back with the tackler's question;
   - `escalate` calls `arbitrate` with the case brief path.
Bound: a wrong `resume` or `goto` costs one re-step, or one move to the current row or the halting entry's routed-from row, per attempt (card 7's check); `tackler_max_spawns` caps attempts per run, and a returning stop key goes to the arbiter.

`arbitrate`:
1. Holds any pause and calls `Arbiter` with the saved envelope, the report and the brief.
   Once it returns, a `spawn` record with settler `arbiter` is written only when the call started a fresh attempt, the same test as the tackler's over the arbiter attempts listed before the call; a refusal, a reused verdict or an attached attempt writes none.
2. A refused spawn (not eligible, off, budget spent, an applied resume left the key unchanged) hands back, quoting the refusal, and at an `awaiting` stop relays the `parent_notice` (D3, D8).
3. After the verdict, it runs `recheck` like the tackler path.
4. A `resume` writes a `restep` record with settler `arbiter` and steps with the attempt number, so `shedverbs.RunUntilStop` consumes it through loom's `ConsumeAttempt` hook.
   A step that refuses with `arbiter_attempt` on its envelope hands back with the verdict path.
5. A `hand-on` hands back with the arbiter's question; at an `awaiting` stop the `parent_notice` is relayed after the outcome (D8).
The arbiter spawns only at the three D6 stops: an awaiting stop with a pending escalation, a tackler `escalate`, and a stop key returning after an applied tackler `resume`.

`resumeOpenAttempt` runs once before the first `Step` (the restart case).
1. It reads the status. For a halted run, it asks `OpenAttempts` for the status's stop key.
2. An attempt is open only when the settle records hold no `hand-back` record for that key later than the attempt's spawn.
   A run whose stop the parent took and then resumed, through `lyx loom resume`, start or batten's binary re-step, finds its attempt closed and steps; it never re-reads an old verdict.
3. For an open attempt it rebuilds the stop from the saved envelope the attempt records and hands it to `tackle` or `arbitrate`, which reuse the attempt's verdict or wait on its live agent instead of spawning.
4. An open, applied tackler attempt for the key goes to `arbitrate` instead of re-performing the `resume`.
Bound: the arbiter's own reuse is the shared path's, unchanged, so a parent-resumed stop that recurs at the same key gets the arbiter's earlier verdict, as the LLM driver's re-run does today.

Tests are rows of `TestGoDriver_Settle`, with fake `Tackler`, `ConsumeTackler`, `Arbiter`, `Snapshot`, `OpenAttempts` and `AttemptSnapshot` columns.
Every agent row that starts a fresh attempt asserts the `spawn` settle record with its settler.
- Tackler outcomes:
  - a transient stop calls `Tackler` once, and its `resume` is consumed then re-stepped with a `restep` record of settler `tackler`;
  - a third `reinvoke` failure, and a `blocked` stop whose verify record does not match, each reach `Tackler`;
  - `hand-back` parks with the tackler's question in the report;
  - `escalate` calls `Arbiter` with the case brief path, and the arbiter's `resume` steps with its attempt number;
  - `escalate` at an `unseeded` error hands back when `Arbiter` refuses it as not eligible.
- Budgets: `tackler_max_spawns: 0` (`errTacklerUnavailable`) and `arbiter_max_spawns: 0` (an off refusal) each hand back with no agent started and no `spawn` record.
  An off refusal at an awaiting stop carrying a `parent_notice` still relays it.
- Moves and tampering:
  - a `resume` whose consume is refused as moved is acted on as a new stop;
  - an orch goto during the tackler wait (a re-read status with a goto entry) discards the verdict, writes the report line naming the change, and re-steps, never reading `tampered`;
  - with the stop key unchanged and the snapshot changed, the hand-back cause is `tampered`;
  - a stop key returning after an applied tackler `resume` goes to `Arbiter`;
  - a re-read status moved to an awaiting stop with a pending escalation calls `Arbiter` with a readable envelope path.
- Restart:
  - a halted run with an open tackler attempt calls `Tackler` before any `Step` and writes no `spawn` record for the reused attempt (the reuse itself is `TestTacklerCall_Rerun`'s);
  - one with an open, applied tackler attempt goes to `Arbiter` and never calls `ConsumeTackler`, writing no `spawn` record for a reused arbiter attempt;
  - one whose attempt a later `hand-back` record closed calls `Step` first and `Tackler` only at the next stop.
- Awaiting escalations: an awaiting stop with a pending escalation calls `Arbiter`, and a `hand-on` parks with the `parent_notice` relayed after the arbiter outcome.
- Pauses: a pause covering the slug starts no tackler and no arbiter until `PauseHold` returns.

**ImpactSummary:** extends the unexported Go driver loop and its one test table with the agent paths; no caller outside the driver changes and the `drive` verb is still unwired.
