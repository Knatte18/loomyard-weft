```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes, step by step, on concrete inputs.

**What I ran:**
- **`firstPassExited`, the `PassExitOriginWait` branch** (`internal/dobbyengine/starts.go:362`): **wrong.**
  - Case: the pass's own catch-up fails for a reason tied to its slug. The dobby's catch-up, which covers only the dobby's standing pairs, then succeeds.
  - Trace:
    - Tick N reads exit 80 and calls `oweCatchUp`; `startReady` returns early.
    - Tick N+1 runs `catchUp` with `catchUpNext` at zero; it succeeds and clears everything (`catchupwait.go:114`) and sets `owesAll`.
    - With no start record, `startReady` spawns the pass again, and it exits 80 again.
  - Result: a restart every couple of ticks, with no backoff, no notice and no wait record, and a full pass owed to every run on each cycle.
  - Case: origin down for the dobby's catch-up too (the test's case): correct.
- **Same branch, when `Deps.CatchUp.PullBolt` is nil** (`catchupwait.go:51`): **wrong.**
  - `oweCatchUp` does nothing, so `catchUpOwed` stays false, and no start record or backoff is written.
  - The next tick's `startReady` respawns the pass at once. That makes an unbounded spawn loop with no notice.
  - The pre-diff code backed this case off and posted a notice.
- **`startReady`'s `catchUpOwed` guard** (`starts.go:94`):
  - Case: owed when the tick began. Unreachable, because `scheduling` is false (`loop.go:731`). Correct, but a no-op there.
  - Case: owed by a first-pass exit read earlier in the same tick. It skips starts. Correct.
  - Case: owed by a later pass's `PassExitOwnedElsewhere` or `PassExitOriginWait` read in `drivePasses`, or by `oweHandOverCatchUp`. Starts are held too; this is consistent with the tick rule.
  - Side effect: `holdReadyBeam` and the `d.busy` update are skipped for that tick. `busy` was already set by the flights loop on earlier ticks, and a ready beam's notice is delayed by only one tick. Harmless.
- **`catchUpFailed`, carrying `Since`** (`catchupwait.go:84-88`):
  - Case: first failure with no record. `Since` is now. Correct.
  - Case: same reason as a record with `Since` set. `Since` is carried. Correct.
  - Case: same reason as a pre-`Since` record (zero). `Since` falls back to `At`. Correct.
  - Case: reasons alternate (origin, then failed, then origin). `Since` resets each time, so the origin wait reads short. Acceptable as documented ("in a row").
  - Case: `UpdateState` fails. The error is logged and the record is left unchanged. Consistent.
  - Case: dobby restart. The record persists and `oweCatchUp` runs at start (`loop.go:510`), so `Since` is carried. Correct.
- **`summarisePushes` with the catch-up wait** (`pushsummary.go:86-99`):
  - Case: ledger empty, no waits, origin wait at 10m against a 10m threshold. `period.Since` is the wait's `Since`, and the notice is posted. Correct.
  - Case: catch-up succeeds after the notice. The record clears, so `owed` and `waiting` are empty and the all-clear is posted. Correct.
  - Case: reason `refused` or `failed`. Not listed. As designed, because those post their own notice.
  - Case: `ReadState` error. It is now returned before the waits are formatted. Equivalent.
  - Case: a beam dobby's catch-up record. It is never read, because the summary runs only on the prime and reads `d.geom.StatePath`. See the handler's scope; no correctness defect in the lines themselves.
- **`waitingSince`**: zero `Since` returns `At`; set `Since` returns `Since`. Correct.

**Findings:**

1. **BLOCKING, design.** Location: `internal/dobbyengine/starts.go:362`, `firstPassExited` case `PassExitOriginWait`.
   - Summary: a first pass whose own catch-up fails while the dobby's catch-up succeeds is restarted every few ticks without bound. It gets no backoff and no notice, and each cycle owes every run a full pass.
   - The two catch-ups cover different slugs:
     - The pass covers only `[slug]` (`internal/battencli/wire.go:1043`).
     - The dobby covers `DobbyPairSlugs`, the standing pairs only (`internal/beamcli/dobbydeps.go:324`, `internal/battencli/catchup.go:340`).
   - So the pass can hit a slug-specific failure the dobby's catch-up never sees. Examples:
     - `FetchRecords` failing, since any fetch failure reads as unreachable origin (`catchup.go:242`).
     - A `ReadRun` error.
     - `PushOwed` or push-lock trouble.
     - `ErrRecordsLayout`.
   - `catchUpSucceeded` then clears the owed flag (`catchupwait.go:114`), and `startReady` respawns the pass because no start record exists (`starts.go:139-146`).
   - Before the diff, this path got a notice and a backoff of up to an hour.
   - This also falsifies the change description's claim that "the dobby's catch-up then classifies the failure itself and posts its own notice … So no case loses its report".
2. **MEDIUM, design.** Location: `internal/dobbyengine/starts.go:362` with `catchupwait.go:50-54`.
   - Summary: when `Deps.CatchUp.PullBolt` is nil, an origin-wait first pass records nothing and owes nothing, so the next tick spawns it again with no limit.
   - The pre-diff path recorded a refused start with a backoff.
   - The doc at `catchupwait.go:49` and the case in `firstPassExited` do not handle "no catch-up wired". It should fall back to `refusedStart` when `oweCatchUp` leaves `catchUpOwed` false.

Tool calls:
- `grep` for `idleCeiling`/`wakeFloor` and `pairteardown.ReadRun`: I needed the catch-up delay bounds and whether a fresh slug's run read can fail, and neither was in context.
