# Round 2 fixer report — discussion

## Changes

- F1 (BLOCKING), fixed differently: D6 gains `lyx drift waive --reason <text>`, which writes a waived mark into the scope record so `Drift-Verify` returns Done without a compare on every later entry.
  D5's deferral and adopt paragraphs, D6's halt and the defer and adopt acceptance criteria name `lyx drift waive` followed by `lyx loom resume` instead of a forward goto, and a waive criterion is added.
  I did not make the proposed refusal "unless the run is halted at `Drift-Verify`": at a failing-compare halt under scope `partial` an arbiter's waive would bypass the parent's withdrawn tests (F7).
  The waive refuses instead unless the run is halted at `Drift-Verify` with no passed capture record, and after the self-check that halt is a decision halt under every scope.
- F2 (BLOCKING), fixed differently: D4 gains a Windows paragraph with `proc.DetachBreakaway`, a named job through `proc.HoldNamedJob` named as `shedrun.LoopJobName` names a loop's, and `proc.TerminateNamedJob` in `pairteardown` and the adopt; the teardown Constraint and an acceptance criterion were updated.
  I did not keep the proposed "falling back as `spawnLoop` does" for the row's capture: no job lyx creates sets `JOB_OBJECT_LIMIT_BREAKAWAY_OK` (`internal/proc/namedjob_windows.go:26` sets only `KILL_ON_JOB_CLOSE`), so a breakaway out of the step's `StartTied` job is refused, and the `proc.Detach` fallback dies with the step, which is the defect itself.
  The step's and the loop's jobs gain `JOB_OBJECT_LIMIT_BREAKAWAY_OK`; where a job lyx did not create still refuses, loom's row captures in the foreground; `lyx drift capture` keeps the `spawnLoop` fallback.
  Scope gains this process plumbing.
- F3 (BLOCKING), as proposed: D4 says the held capture lock and a mechanical compare's marker count as activity in `agentActivityFor`; D5's waiting bullet names the watchdog; the loom waiting criterion covers a capture longer than `step_idle_timeout_min`; the compare marker joins the Constraint's `.lyx/drift/` list.
- F4 (BLOCKING), as proposed: under scope `full` every `Drift-Verify` halt but an error or transient stop carries `DecisionHaltPrefix` (D6); a caller's `lyx drift accept` under `full` refuses outside a decision halt (D7); D2's arbiter lines restated; an acceptance criterion added.
- F5 (BLOCKING), as proposed: D2 says all three Drift rows carry `entry_gate: drift.enabled`; the first acceptance criterion says every Drift row is `Skipped`; the support log drops the rejected alternative and marks Q6 superseded.
- F6 (MEDIUM), as proposed, and tightened in the self-check: D5 states the flag-off outcome, with a criterion.
- F7 (MEDIUM), fixed differently: a caller's accept of a withdrawn test refuses outside a decision halt (D7).
  I did not make "a failing final compare whose failing tests include a withdrawn one" a decision halt: the row cannot tell which tests failed, since the compare tool's output format is out of scope (Scope, Out).
  Under `partial` every failing final compare while the withdrawn list is non-empty is a decision halt instead (D6); the criterion is updated.
- F8 (MEDIUM), fixed differently: D6 names the real bound, but as `tackler_max_spawns` and `arbiter_max_spawns` in `shed.yaml` (`internal/sheddriver`), counted per run.
  The proposed "cap on arbiter resumes of one halt" does not exist: `internal/sheddriver/template.yaml:7` counts arbiter spawns per run, and the tackler may also resume a blocked stop (`internal/sheddriver/doc.go:41`).
- F9 (MEDIUM), as proposed: D3 gives darn's failed-capture Stuck both ways forward; D5 widens `lyx drift defer` to `Drift-Baseline` on a failed or refused capture, after which the row returns Done; the darn criterion is updated.
- F10 (MEDIUM), as proposed, ordered so `awaited_at` set returns Done before any capture is tried: D3 states the absent record, the refused capture and the missing `baseline.md`; a criterion is added.
- F11 (MEDIUM), as proposed: the Constraint is rewritten, and the .NET `bin`/`obj` risk moved to Open risks naming `csharp-sandbox-hub`.
- F12 (LOW), as proposed: D2 separates a landing's merge from loom's Finalize reroute.
- F13 (LOW), as proposed: D4's commit rule covers every way out of `Drift-Await` and leaves stray capture logs to `Drift-Verify`.
- F14 (LOW), as proposed: the first Open risks bullet is dropped; Q1 is updated.
- F15 (LOW), as proposed: the note now says darn's build already uses the filled env.
- F16 (LOW), as proposed: Interview turn 3's interactions 4 line points at D5 and bound-breaker 3 at Q12; Review rounds has entries for round 1 and round 2; Q12 is corrected.
  Round 2's picks are recorded as Q16 to Q19.

## Self-check

- fix-consistency, LOW, D4's caller-capture sentence names only `Drift-Await`: fixed as proposed, it also names a `Drift-Baseline` halt on a failed or refused capture.
- fix-consistency, LOW, D2's reroute sentence ignores a waive: fixed as proposed.
- fix-consistency, LOW, support-log goal-scope 1 and goal-scope 3 lines point at removed text: fixed as proposed, both marked superseded.
- fix-consistency, LOW, the waive and withdrawn-accept refusals name no way forward: fixed as proposed.
- fix-way-forward, MEDIUM, loom's Windows foreground Stuck names no way forward: fixed as proposed, it carries darn's ways forward, with a criterion.
- fix-way-forward, LOW, defer cannot tell a refused-capture halt: fixed as proposed, through a fixed capture-halt marker in the halt reason.
- fix-way-forward, LOW, D4 describes only the `Drift-Await` caller: merged with the first fix-consistency finding.
- fix-claims, MEDIUM, "every other descendant of a step still dies with it" ignores `spawnLoop`'s breakaway: fixed as proposed, the exception is named and left to the plan to check.
- fix-proposal, MEDIUM, flag off then on compares against an unawaited capture: fixed as proposed, loom's `Drift-Verify` counts a capture as passed only once `awaited_at` is set.
- fix-proposal, MEDIUM, a watchdog lock probe can make a starting capture refuse: fixed as proposed, a probe releases at once and a capture start retries briefly.
- fix-proposal, MEDIUM, an arbiter can defer and waive under `partial`: fixed as proposed, the no-passed-capture halt is a decision halt under every scope.
- fix-proposal, MEDIUM, a reviewer's plain build takes no slot: fixed as proposed.
- fix-proposal, LOW, nothing enforces the respawned agent writing `baseline.md` alone: fixed as proposed, `lyx drift scope` refuses once a scope record exists.
- fix-proposal, LOW, the dirty-tree refused-capture Stuck names only defer: fixed as proposed, it also names cleaning the tree and resuming.

## Disputed

None.

## Deferred

None.
