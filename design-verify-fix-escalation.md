# Webster verify failures the fixer fork cannot clear escalate to an Opus-high fixer

`verify-fix-escalation` · Note · enhancement, webster, loom

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

## Related

The `loom-tail-review` task makes the Merge-In fixer one Opus agent; this note is about Webster's own gate, before Merge-In.
