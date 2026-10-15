# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed), `_lyx/plan/02-shuttle-gate-spans.md`: the **finalize:** bullet now splits the close around `evaluateGate(true)`.
  Before the evaluation, on a Done outcome with a gate, `finalize` calls `endReworkSpans` with `result.EndedAt`, which `finalize` reads before the evaluation (confirmed at `internal/shuttleengine/wait.go:1605`), so the closed rework span lands before the final evaluation's gate span.
  After the evaluation, `closeGateSpans` ends a held span still open and stamps the copies.
  The test bullets are unchanged: the deadline and session-lost rows read gate, rework, gate, and the held-at-deadline test reads gate, held.
- **F2** (fixed as proposed), `_lyx/plan/13-role-prefix-config.md`: added `TestBouncer_ReBounceProbesForALiveSeed` and `TestBouncer_EntryProbe_SpecNamesTheJudgesOwnOutputFiles` under **Edit:**.
  A new test bullet has each one set `RolePrefix: "plan"` on the fixture's `Config` before `Build` and assert `GotAttachSpec.Role` is `plan-bouncer-seed` or `plan-bouncer-judge`.
  The `TestBouncerEntry_EnvJudgeFallback` bullet now names the Attach that `runSeedSpawn` and `judgeCall` run on their own spec (confirmed at `bouncer.go:1236` and `bouncer.go:1373`), not the probes.
- **F3** (fixed as proposed), `_lyx/plan/06-prrework-carry-gate.md`: `TestPRRework_ClassRecorded` gains a parse-failure row.
  Its fake session leaves a plan `planparser.ParsePlan` refuses, and the row asserts Stuck, the `rework plan does not parse` reason prefix, and the fixture's gate count and spans on the pointer.
- **F4** (fixed as proposed): card 3's `GateSpan` stores `Start` and `End` as `time.RFC3339Nano` UTC, and card 5's `gateSpansOf` formats with `time.RFC3339Nano`.
  Card 10 is unchanged, since Go's `time.RFC3339` layout parses fractional seconds.

## Disputed

None.

## Deferred

None.
