---
round: 2
focus:
  - "In `_lyx/plan/02-shedengine-relaunch.md`, check that every refusal `Relaunch` adds (missing status file, done run, running run) has a `contracts/specs/refusal-spec.md` row landing with it and a test row in `TestRelaunch` that reaches it; settle whether `Goto`'s refusal precedence among unknown target, running state and admission is stated where `CheckGotoTarget` is called."
  - "In `_lyx/plan/10-arbiter-verb.md`, check that each refusal the verb adds (missing status file, unreadable envelope, status read error) has a named test row, and that the re-run path states what happens when the marker's deadline has passed (shuttle refuses a negative `Timeout` and reads zero as the config default) and when `AttachIfLive` errors."
  - "In `_lyx/plan/12-relaunch-consume-attempt.md`, check that every refusal of `loomConsumeAttempt` and `consumeArbiterAttempt` (non-positive or non-numeric attempt number, unparseable verdict) is reached by a named test, and settle whether consumption of a `resume` rests on the verb's validated result or only on the agent-written `verdict.yaml`, so the card's stated bound is true."
  - "In `_lyx/plan/14-driver-arbiter-branch.md`, check that the driver's re-run wake rule is bound to the decision record's condition (an unapplied attempt for the current stop key) rather than to every loop-job exit, and that the `**Intent:**` states what the new routing edges can skip and what bounds them."
  - "Across `_lyx/plan/09-*.md`, `_lyx/plan/10-arbiter-verb.md`, `_lyx/plan/03-shedverbs-arbiter-attempt-flag.md` and `_lyx/plan/12-relaunch-consume-attempt.md`, check per `PATTERN-test-economy` that no planned test re-covers behavior another card's test or an existing test already pins."
---

Focus targets the sites where round 1 found defects, so the fixer's changes get a fresh read.
