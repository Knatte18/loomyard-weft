```yaml
lens: fix-claims
outcome: reported
```

fix-claims: I checked the claims the fix added or rewrote in `_lym/discussion/decision-record.md` against the code at HEAD.

**Claims that hold:**
- `MayHold` lets a must-pass entry take `Final` and changes nothing else (`gate.go:74-78`, `wait.go:1625`).
- A goto appends a history entry (`goto.go:135`, `goto.go:249`).
- A folded call increments the newest entry's `Repeats` (`run.go:703`).
- `shedrun.StatusFile` exists.
- `registerRunner` registers under `originSlug` and skips the prime and anything outside a hub (`detachfront.go:807-818`).
- The parent directive has a no-parent variant (`websterengine/render.go:372`).
- `darn-write` and `card` are real gate.yaml keys (`gateslot/template.yaml:19,26`).
- The new deadline bound of one fixer timeout plus three inner runs holds against the re-prompt, spent-budget and pass paths.

**Findings:**

1. **MEDIUM.** D2, new sentence: "A finalize after a boundary evaluation reads that evaluation's memoised verdict and runs no closure (`evaluateGate`, `internal/shuttleengine/wait.go`)."
   - **Summary:** This is true only when the boundary evaluation itself finalizes. After a re-prompt, or after a held re-prompt send, the memo is cleared, so a later finalize runs the closures again.
   - **Evidence:** `wait.go:583` and `wait.go:557` both set `run.gateVerdict = nil`. Only the pass, terminal or spent-budget path finalizes on the memo (`wait.go:536-539`). The fix's own deadline bound depends on that later finalize running the closures through `Final`, so the sentence contradicts its own bound.

2. **MEDIUM.** D2 Bound, rewritten: "so at most the entry's `attempts` plus one fixers per attach of the writer's run, one per boundary failure the budget allows".
   - **Summary:** A held re-prompt send spends no attempt and clears the memo, while the writer stays idle at the boundary. The next tick evaluates again and the wrap spawns a fresh fixer at an idle boundary, so the fixer count is not bounded by `attempts` plus one.
   - **Evidence:** `wait.go:546-558`. On `ErrSessionBusy`, `ErrSubmissionNotLanded` or `ErrPaused` the loop returns without `run.gateFails[failed]++` (`wait.go:567`), sets `run.gateVerdict = nil`, and re-sends "on a later tick". With the memo cleared, `handleGatedBoundary` runs `evaluateGate(false)` again (`wait.go:516`).

3. **LOW.** D1, rewritten: "so pair teardown, `lym batten cancel` and tender's reaping, the paths that call runproc's `End`, `EndSparing` or `EndRole` today, stop it".
   - **Summary:** The sentence presents this as the complete list of runproc callers, but it is not.
   - **Evidence:** `fabriccli/anchordeps.go:243` calls `runproc.EndSparing` on the worktree's owner and ends every role it does not spare, which would include a fixer. The orch, beam and pawl paths call `EndRole` too (`orchcli/cli.go:263`, `loomcli/pawlwiring.go:274`). Naming the group instead of listing it would avoid this.

4. **LOW.** D1 and D4, new text: "the owner `originSlug` names".
   - **Summary:** `originSlug` is unexported in `internal/gatecli`, so the row callers in `internal/gatefixer` and `internal/shedrecipe` cannot call it.
   - **Evidence:** `internal/gatecli/run.go:363` declares `func originSlug`. It reads `fabricengine.ReadOrigin` and returns `location.WorktreeName`, and the record should name that source instead.

Further tool calls:
- Read `shedengine/run.go:860-900`, grepped the `Repeats` fold and `originSlug`, and listed the gate template keys, to check the D8 visit-key, D1 and D3/Scope claims; my context did not have those spans.
- Read `gateslot/template.yaml` keys, `battencli/cancel.go`, `fabriccli/anchordeps.go:235-250` and `gatecli/run.go:355-375`, to check the D1 caller list and the `originSlug` claim, which my context did not hold.
- Read `wait.go:544-585`, to check the D2 memo and fixer-count claims against the re-prompt paths my context lacked.
