## Goal

```markdown
Imported from [issue #51](https://github.com/Knatte18/loomyard/issues/51).

What happened: the run halted as failed at step Webster with anomaly interrupted-stop. The status error was `loop-exited: loop ... ended without an envelope; its output is in .lym/shed/<task>/steps/loop.log; step Webster was in flight under trace ...`, followed by `shedverbs: step Webster interrupted (loop-exited)`.
The way forward named was to read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, and re-run `lym shed step <task> --until-stop`.

Why it is abnormal: a loop must end by writing an envelope. The note records no cause for the missing envelope (no crash, signal or budget reason), and the preceding history shows all earlier steps through Drift-Await finishing done, so the exit came while Webster was starting. The halt note carries nothing that lets the operator tell a loop crash from an external kill.

What to do: have the loop-exited anomaly record the exit cause (exit code or signal, last lines of loop.stderr_path) in the halt note itself. Also check whether a loop can exit between Drift-Await done and the Webster start without writing an envelope, and make that path write one.

Provenance: task server-based-live-viewer; notes loom-halt-2.md.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-05): A signal ends the loop without an envelope; a panic writes one

Decision: Accepted: the entry's ask to make a no-envelope exit write one is met for the loop's own failure path (a panic is recovered and ends in an interrupted stop with its envelope), while a terminating signal only records a mark in the loop log and the next waiter's dead-loop stop reports it with the signal named. Review finding F3 is settled by this decision; the code stays as built.

Rationale: SIGTERM reaches the loop from runproc.End on a cancel, a teardown or a reap; writing a failed stop inside the dying loop's kill grace would file halt notes for deliberate cancels and race the teardown's mark. The entry's goal, telling a crash from an external kill, is met by the recorded exit cause.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - no-starting-mark-untested (class scope, severity MEDIUM, open in rounds 1)
  - panic-recovery-narrower-than-claims (class design, severity MEDIUM, open in rounds 1)
  - signal-choice-unrecorded-decision (class decision, severity MEDIUM, open in rounds 1)
  - stale-step-stderr-on-dead-loop-stop (class design, severity MEDIUM, open in rounds 1)
  - trace-copy-untested (class scope, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
