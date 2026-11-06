```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer applied as proposed (F1–F5) through the fix-proposal lens.

**Findings**

1. **MEDIUM**: `internal/shuttleengine/loadturn.go`, the `else if watch.open` branch that F1 asked for.
   - **Summary:** an interrupted load turn now holds the start until the run deadline.
   - **Evidence:**
     - `watch.open` clears only on a turn-end or session-start signal or a stop event.
     - `doc.go` says a provider's interrupt is a separate marker: "the interrupt marker turns a turn that is still open into an interrupt stall". `loadSkillTurn` never consults it, and the engine seam has `TurnStartInterrupt`.
     - The case: the operator presses Esc during the first load turn and the provider writes a turn start but no turn end. `open` stays true, the prompt is never typed, and the pane is live, so the died check never fires. The start sits until `run.deadline`, which can be an hour or more. Before the fix, the same case skipped after the open bound.
   - The proposal's "keep polling until a turn end is read, the pane dies or the deadline passes" omits an interrupt exit. Nothing pins or documents the case.

2. **MEDIUM**: `internal/shuttleengine/loadturn.go` and `abandonLoad`, the deadline exit for an open turn.
   - **Summary:** a slow, healthy load turn can now fail the whole start as `timeout` where it used to skip a skill.
   - **Evidence:**
     - The deadline is `createdAt + Spec.Timeout`, and callers choose `Spec.Timeout` freely. The default is `run_timeout_min`, which is 5 in `fastConfig`.
     - A caller with a short run timeout and a slow load turn, such as a first turn after a cold start, now gets `abandonStartup(OutcomeTimeout)` with the strand removed. Before the fix, the same turn gave a skipped-skill warning and the prompt was typed.
     - `Spec.SkillLoadTimeout`'s comment says only that the turn is "awaited until it ends or the run's deadline passes". It does not say the start fails.
     - No test covers an open turn with a deadline shorter than `SkillLoadTimeout`.

3. **LOW**: `internal/shuttleengine/startup_test.go`, the "turn start read during the back-off" row and `Carries{0: {startAfter: 2, endAfter: 3}}`.
   - **Summary:** the row passes only because startup consumes exactly one `Status` call.
   - **Evidence:**
     - The values are tuned to the `skillReed` comment about `Dies` ("the startup probe's single ready tick").
     - If startup gains or loses a `Status` call, `START` lands in the same read as the API-error end, or after the retype.
     - The first case leaves the row passing without testing the cancellation. The second makes it fail with a typed load.
     - The coupling is not stated at the row or on `carriedTurn`.

4. **LOW**: `pattern/PATTERN-reed-panes.md`, the new `Rule:` line for the API-error hold, which ends in `(test)`.
   - **Summary:** the rule says "never typed", but the Bound beneath it says an attach delivers a pending prompt without judging the API.
   - **Evidence:** the Bound sits below the rule. The rule itself should not claim more than the test pins, which is the `Start` path only.

No further finding on F4's prose or on the F5 daemon-wakeups entry.

**Tool calls:** none. My context already held every file I needed.
