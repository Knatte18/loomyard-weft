# Support log — cold-gate-fixer

## Interview

Autonomous mode: no operator answers; every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — the board and the code

Read the task entry, the board list, project `turn-savings` and the touching notes E20 `fixers-on-sonnet`, R1 `agent-turns-effect`, R2 `planner-fork-lookups`, E1 `finalize-fix-outside-lock` and E7 `batching-breach-nudge`.
No `Decision for cold-gate-fixer:` note exists, and `lym loom peers` lists no peers or parent commits.
None of the touching entries overlaps: E20 is about review fixers, R1 measures this task after it lands, R2 and E7 cut other writer turns, E1 concerns Finalize's merge fixer.

Read webster's cold fixer (`coldfixer.go`, `verifyfixer.go`, `fixverify.go`, its two stencils), the shuttle gate contract (`gate.go`, `evaluateGate` in `wait.go`), the recipe gate resolver (`entries_gate.go`), Darn-Write's producer and entry, the gate.yaml template, the darn, tapestry and loom templates, the Webster card gate renderer and the writer stencils' self-check text.

### Turn 2 — where the row fixer hooks in

Three ways: block inside a wrapped gate closure, return pending and hold the run through `MayHold`, or have the writer run a fixer verb.
Picked the blocking wrap: no shuttle change, and the verify gate already blocks the same way.
Pending was rejected after reading `evaluateGate`: each re-evaluation starts from the first entry, so Darn-Write's clean gate would fail on the fixer's uncommitted edits and re-prompt the writer mid-fix.

### Turn 3 — the card gate

The card gate is a fork's own `lym gate test --gate card --card <id>` call, not a row gate.
Picked a hand-off inside the agent gate call's detached runner, keyed by a `fixer` field of the gate key, so the gate verb learns nothing of webster and the fork waits on the call it already makes.

### Turn 4 — config, model, line, writers

- Config: rows name a `<module>.<key>` setting from their recipe gate element; the agent gate call reads its gate key's `fixer` field; one `fixer_timeout_min` in gate.yaml.
- Default model `sonnet[medium]` for all four callers.
- The fixer gets every failure and leaves design-changing ones by its prompt rule; Go's re-run hands the rest to the writer.
- Plan and Discussion writers stop looping on their validator when the fixer is on, or the fixer would never run for them.
- Webster's `fix-verify` keeps its trigger and ladder and moves onto the shared core and frame.

PATTERN entries checked against the design, none changed but the one added: `PATTERN-interactive-spawn`, `PATTERN-shuttle-provider-seam`, `PATTERN-parent-directive`, `PATTERN-stencil-ownership`, `PATTERN-agent-name`, `PATTERN-shed-recipe-registry`, `PATTERN-no-hard-coded-choice`, `PATTERN-config-layers`, `PATTERN-gate-slot`, `PATTERN-keep-warm-cap`, `PATTERN-durable-vs-ephemeral-state`, `PATTERN-tolerant-records`, `PATTERN-daemon-wakeups`, `PATTERN-friction-capture`, `PATTERN-run-process-owner`.

### Turn 5 — lens review

Six forks returned reports; none was missing.

- existing-paths, walkthrough, bound-breaker (BLOCKING, merged): the card fixer's own gate call shares the detach key of the run waiting on it and deadlocks. Fixed in D4: a fixer strand's call keys its detached run with its strand name.
- existing-paths (BLOCKING): Plan-Write sets `Forked` after the shared resolver, so a wrap there misses the forked plan gate every default run evaluates. Fixed in D2 and the plan notes: the wrap applies to the final closures, after the row constructor; a fact added to Constraints.
- existing-paths (MEDIUM): an earlier entry's pass goes stale after the fixer. Rejected for the three rows: Darn-Write's verify gate fails on a dirty tree itself, as interactions and bound-breaker confirmed; D2 now states the rule that keeps it so.
- existing-paths (MEDIUM): an orphaned row fixer outlives cancel and goto. Fixed in D1: the strand is registered with runproc under its run.
- walkthrough, bound-breaker (MEDIUM, merged): after a driver restart nothing re-evaluates until the deadline. Fixed in D2: the wrapped entry's `Expired` probe finalizes an attached run holding a fixer record; a fact added to Constraints.
- walkthrough (MEDIUM): a fixer that changes the dirty paths makes the fork's next call start a second run. Fixed in D4: a run in its fixer or re-run phase is joinable whatever the tree identity.
- bound-breaker (MEDIUM): slots held across the fixer wait. Fixed in D2 and D4: no slot is held while the fixer works.
- interactions, walkthrough (MEDIUM, merged): D1 and D6 both claimed the record. Fixed: the store is a caller seam, webster's is its `VerifyFixers` state.
- interactions (MEDIUM): webster's timeout. Fixed in D3 and D6: `fixer_timeout_min` serves the four callers, webster keeps `recovery_timeout_min`.
- interactions, walkthrough, bound-breaker (MEDIUM/LOW, merged): the Discussion fork boilerplate and the lens procedure's validator loop were left. Fixed in D7.
- interactions, bound-breaker (LOW, merged): webster's `verify-fixer` role and recovery strands against the D4 guard. Fixed in D4 and D6.
- interactions (LOW): fixer-on test read from the setting alone. Fixed in D7: read from the resolved gate entry.
- bound-breaker (LOW): deadline overrun bound and the attempts bound across attach. Fixed in D2.
- existing-paths (LOW): Darn-Write's kept findings carry the fixer text. Fixed: one clause in D2.
- board-scope (LOW): D5's wording against the entry's "from what the reports contain". Fixed in D5's rationale.
- board-scope (LOW): why Darn-Write's clean and description gates stay with the writer. Fixed in Scope Out.
- leanness (MEDIUM): the rejected pending alternative in D2. Fixed: moved to the Constraints fact, deliberation kept here.
- leanness (LOW): duplicated facts in D3, D4 and Constraints, the pause risk, the Scope bullets, the PATTERN list, the Goal restatements, "Tests for the above". Fixed: each kept once; the PATTERN list moved to Turn 4 above.

## Rejected alternatives

- Pending plus `MayHold` for the row fixer: re-evaluation restarts from the first entry, so an earlier clean gate fails on the fixer's edits mid-fix.
- A writer-invoked fixer verb, as `fix-verify` is for Merriam: it costs writer turns at full context, which is what the task removes.
- A Go-side table sorting findings by code into fixer and writer classes: it duplicates each spec's list of checks and goes stale; the fixer's prompt rule and Go's re-run give the same split.
- The card fixer spawned by Merriam after the fork reports `FAILED`: the fork's session would end and its finished cards would need a recovery strand.
- A `lym webster card-gate` verb wrapping the gate call: it would rebuild the capped, detached gate call inside webster.
- All fixer models in gate.yaml: the plan and discussion gates are not gate.yaml keys, and their rows already read settings by key.
- Haiku as the default: too weak for test failures; `sonnet[low]`: no evidence it suffices, and the model-spec is one edit away.
- Converting Webster's plan-level verify gate into a row-style wrap: it would change its fork-first ladder, which the board does not ask for.
- Covering `lym gate card-verify` lines: they run after the card's commit and are arbitrary commands, so the fixer's commit rule would differ again.
- Moving webster's `fix-verify` onto `gate-fixer` and `fixer_timeout_min`: it would change its tokencount identity and its timeout, which nothing asks for.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): where does the row fixer hook in? Blocking wrap of the gate closure (D2).
- Q2 (auto-pick, resolved): how does the card gate reach the fixer? Through the agent gate call's detached runner, keyed by the gate key's `fixer` field (D4).
- Q3 (auto-pick, resolved): where do the model keys live? Recipe element naming a module setting for rows, gate key field for the agent call, shared timeout in gate.yaml (D3).
- Q4 (auto-pick, resolved): default model? `sonnet[medium]` (D3).
- Q5 (auto-pick, resolved): which failures go to the fixer? All, with a prompt rule to leave design-changing ones; Go's re-run returns the rest to the writer (D5).
- Q6 (auto-pick, resolved): how is `fix-verify` generalised? Shared core and frame, same trigger, ladder, role, timeout and records (D6).
- Q7 (auto-pick, resolved): do writers keep their validator loops? No, when the fixer is on (D7).
- Q8 (auto-pick, resolved): how many fixers per failure? One per failed evaluation or red call (D2, D4).
- Q9 (auto-pick, resolved): are `card-verify` lines in scope? No.
- Q10 (auto-pick, resolved): what happens on a driver restart mid-fixer? The `Expired` probe finalizes and the finalize evaluation waits on the fixer (D2).
