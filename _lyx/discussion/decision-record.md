# Decision record — verify-fix-escalation

## Goal

A plan-level verify failure in Webster that the in-session fixer fork does not clear goes to a cold fixer at the recovery role (`opus[high]` by the template) before the run halts, instead of every fixer attempt being a fork on Merriam's own model.

## Scope

- `internal/websterengine`: the verify gate routes each failure to a fixer, a new spawn-and-wait engine for the cold fixer strand, its state record, the reclaims that stop it, the prompt render, the template default of `verify_gate_attempts`, and `doc.go`.
- `internal/webstercli`: a new role-audience verb `lyx webster fix-verify` and its wiring.
- `contracts/stencils`: the verify-fix stencil split into a shared body and two prefixes, the cold prefix registered as a spawned role's opening stencil, and the Master template's gate-failure section.
- Out of scope: the tackler, the arbiter, the driver, the Merge-In fixer and the batch recovery path.

## Decisions

### The gate routes each failure to a fixer

Decision: the verify gate decides, at each failed evaluation, which fixer the failure goes to, and records it as `fixer: fork` or `fixer: cold` in the verify-gate report.
A failure goes to the cold fixer when it is the last failure the attempt budget re-prompts (its failure count equals the cap), when an earlier failure of the same shuttle run already went cold, or when the previous fixer added no commit above the pre-fix head.
Otherwise it goes to the fork.
The findings' way-forward line and the re-prompt line name the routed fixer: the fork step with the prompt file as today, or the backgrounded `lyx webster fix-verify` call.
Rationale: the escalation has to happen inside the gate, before the run halts, since nothing after the halt fixes code; the gate already owns the attempt count and the fix-commit range, so it is the one place that can tell the last attempt and a fixer that committed nothing.

### One fork attempt, then one cold attempt, by default

Decision: the `webster.yaml` template's `verify_gate_attempts` drops from 3 to 2, so an untuned hub runs one fork fixer and then one cold fixer before the gate stops the run.
Rationale: the operator's view is that the fork's model is often too weak for these failures and three attempts are more than needed; with the routing rule above, a cap of 2 gives exactly one of each, and a hub that tunes the cap keeps the cold fixer as its last attempt.

### A blocking verb spawns the cold fixer

Decision: `lyx webster fix-verify` spawns the cold fixer as its own strand at the recovery role (`RoleRecovery`'s model choice, the `recovery` key), under strand role `verify-fixer`, with the cold verify-fix prompt, and blocks until it ends or `recovery_timeout_min` passes, as `recover-batch` does.
It is re-entrant: a call that finds a recorded fixer that is not terminal attaches to it and waits.
It reports `done` when the fixer wrote its reply file, and `dead` with a reason when its turn ended without one, its strand is gone or the timeout passed; every terminal call stops the strand, and only `done` removes its run dir.
Its envelope carries the status and the reply text; a non-done status exits non-zero.
Merriam runs it backgrounded, ends its turn and acts on the completion notification, so the verb joins the Master spawn's awaited shell prefixes beside `recover-batch`.
Bound: the verb can spawn a recovery-role strand whenever a verify-gate report exists, also after a failure the gate routed to the fork, and it counts no spawn of its own.
It refuses with no verify-gate report on disk, and it attaches rather than spawning a second strand while one is recorded live.
How many cold fixers a run gets is bounded by the gate messages Merriam receives, which the attempt budget caps, and by the Master template's rule to run it only when a gate message names it.
The cold fixer's commits pass through the same gate commit check, verify, run-exit audit and Webster-Review as the fork's.

### State records every cold fixer spawn

Decision: `State.VerifyFixers` records each spawn in order: strand guid, session id, engine, spawn time, events path and offset, run dir, and a terminal status once the verb classifies it.
Each recorded session joins the run's write evidence beside the batch recovery sessions, the run-entry reclaim stops a non-terminal recorded fixer, and `reset --to start` stops every recorded fixer strand before it resets.
Rationale: the fixer is a separate process writing in the worktree, so the reclaims and the evidence that cover recovery strands must cover it the same way.

### The verify-fix prompt becomes a shared body with two prefixes

Decision: `webster-body-verify-fix` keeps the findings and the fixing rules shared by both fixers; a new `webster-prefix-verify-fix-fork` carries the fork's in-session framing and its reply-as-final-action, and a new `webster-prefix-verify-fix-cold` carries the cold framing, the parent, edit, PATTERN and friction directives, the orientation steps and the reply-file final action.
The cold prefix is the opening stencil of the `verify-fixer` role in the stencil registry's role map.
The cold fixer's reply file sits in Webster's scratch dir, outside `_lyx`, and is the strand's shuttle output file.
Rationale: one statement of the fixing rules for both fixers, the way the implementer body serves the fork and recovery prefixes, while each prefix states only what differs.

### Merriam handles the cold fixer's completion like the fork's

Decision: the Master template's gate-failure section says that when the way forward names `lyx webster fix-verify`, Merriam runs it backgrounded instead of spawning a fork, and on its completion notification, whatever its exit, rewrites its two contract files and ends its turn, so the gate verifies again.
Rationale: the gate, not Merriam, judges whether the fix worked; a dead fixer simply leaves the failure for the gate to count.

## Constraints

- `PATTERN-stencil-ownership`: the new prefixes are registered embedded defaults read at call time through `stencilstore`.
- `PATTERN-parent-directive` and `PATTERN-edit-directive`: the cold prefix renders both markers, and the role map names it.
- `PATTERN-role-skills-typed`: the cold fixer's skills go on its launch spec, as the recovery strand's do.
- `PATTERN-shuttle-stop`: the strand is stopped only through the stopper seam.
- `PATTERN-told-geometry`: the engine derives no path; the reply path is built from the told scratch dir.
- `PATTERN-tolerant-records`: `state.json` gains an optional key, and a state without it loads.
- `PATTERN-refusal-way-forward`: the verb's refusals name the way forward, and a test reaches each.
- `PATTERN-test-economy` and `PATTERN-test-speed`: tests extend the existing gate and render tests, with injected clocks.
- Docs land in the same commit: `internal/websterengine/doc.go` and the CLI verb's long help.

## Auto-mode assumptions

- The operator view in the board entry is adopted as the default: one fork attempt, then one cold attempt.
- The failure count used for routing is the closure's own count of failed evaluations in one shuttle run, so an attach restarts it like the attempt count.
- The cold fixer uses the existing `recovery` model key and `recovery_timeout_min`; no new `webster.yaml` key is added.

## Open risks

- A re-prompt the shuttle could not deliver is re-evaluated on a later tick, which counts one more failure in the closure, so routing may go cold one attempt early; the cap still bounds the run.
- The cold fixer starts from no context and only the report and the plan overview, so a failure whose cause needs the batch history may still beat it.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/darn/round-2-review.md; fixer report: _lyx/reviews/darn/round-2-fixer-report.md.
  - reclaim-leaves-cold-fixer-record-nonterminal (class design, severity MEDIUM, open in rounds 2)
  - shared-fixer-body-claims-unapplied-run-exit-audit (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- With a cap of 2, the first failure routes to the fork and the second to the cold fixer, as the report's `fixer` field, the findings and the re-prompt line show.
- A failure after a fixer that added no commit routes to the cold fixer below the cap, and routing stays cold once it went cold.
- `fix-verify` refuses with no verify-gate report and names the way forward.
- `fix-verify` spawns at the recovery role with the cold prompt, the reply file as its output file and the role's skills, and records the spawn in state.
- The wait classifies done on the reply file, dead on a turn end without a reply, a gone strand and the timeout, and stops the strand at every terminal.
- A recorded fixer session is part of the run-write evidence, a live recorded fixer is stopped at run entry and by `reset --to start`.
- The rendered cold prompt carries the parent, edit, PATTERN and friction directives, the report path and the reply path; the fork prompt keeps its content.
- The template's `verify_gate_attempts` is 2.
