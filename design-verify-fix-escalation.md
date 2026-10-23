# Webster verify failures the fixer fork cannot clear escalate to an Opus-high fixer

`verify-fix-escalation` · Task · enhancement, webster, loom · running · Darn-Burler-Fix

## Seen (orch, 2026-10-23)

When Webster's plan-level verify gate fails, Merriam spawns a fixer fork (`internal/websterengine/doc.go`, The verify-gate fixer fork).
A fork always runs on Merriam's own model, `sonnet[medium]` by the `webster.yaml` template, and Merriam may spawn nothing but forks.
So every one of the `verify_gate_attempts` (3) fixes is a Sonnet-medium fork.
After the last attempt Webster ends stuck; the Webster row has no `on_stuck` edge, so the run halts to the driver and the arbiter, which diagnose and resume or hand on, but fix nothing.
The one stronger model in Webster, `recovery: opus[high]`, is spawned only by `recover-batch` for a batch fork that reports stuck, never for a verify failure.
In `plan-card-rules` the operator asked Merriam to spawn an Opus-high agent if its fixer failed, and Merriam answered that it cannot.

## Idea

A verify failure that the fixer fork does not clear escalates in model before the run halts:
for example, the last attempt, or the attempt after a fork that made no commit, goes to a cold fixer at the recovery role (`opus[high]`), spawned as its own strand the way `recover-batch` spawns one, with the verify-gate report as its brief.

## Operator view, not a decision (2026-10-23)

Sonnet medium is often too weak to find and fix such failures, and three attempts are more than needed: one Sonnet fixer attempt, then the Opus-high fixer.

## Why not the tackler and the arbiter (orch, 2026-10-23)

With the Go driver (`driver_kind: go` in `shed.yaml`), a stuck Webster goes to the tackler (`haiku`) and then the arbiter.
Neither fixes code: the tackler's verdict is `resume`, `hand-back` or `escalate` (`internal/sheddriver/doc.go`), so at best it re-runs Webster with the same Sonnet fixer.
The escalation in model has to happen inside Webster's gate, before the run halts.
The gate already reruns a failure once before any fixer, so a flaky test does not use up the one fixer attempt.

## Related

The `loom-tail-review` task makes the Merge-In fixer one Opus agent; this note is about Webster's own gate, before Merge-In.
