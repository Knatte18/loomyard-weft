```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I attacked the record's claims, bounds and guarantees with concrete counterexamples.

**Findings**

1. **MEDIUM, D3 (restart keeps the first ask and deadline).** The record does not say what key the ask record uses. `assistPaths` names its files by `opts.Round`, which is the attempt token, so "a retry's files are never the first attempt's". If the parent ask is modelled on the assistant seat, as Notes for the plan writer suggests, the ask record would be keyed by that token too. A restarted review row in the same round would then find no ask record, mail the parent a second `[discussion-review request]` and start a fresh deadline. The fix: state that the ask record is keyed by the perch round number, not the attempt token. Evidence: the `assistPaths` doc says "Each name carries the review's stem, the token and the seat, so a retry's files are never the first attempt's".

2. **MEDIUM, D4 and D5 (state transitions).** `parent-review answer`, run by the orch's fork, and `parent-review await`, run by the chair, both write the ask's state. Neither decision says the writes are serialized. Counterexample: the answer verb reads the ask as open at T−1s, await marks it `timeout` at T, and answer then writes `done` with its report. The chair has already consolidated without the parent, but the report is still copied beside the review (D4), and the parent believes it was heard. The fix: one lock or compare-and-set over the ask's state, and the answer verb refuses when it loses.

3. **MEDIUM, D7 ("fix-class finding").** A review finding's class is one of `design`, `scope`, `decision` or `consistency` (rubric "Finding class"; `burler-review-format.md` has a `class:` field). "Fix-class" is not one of them. D7 must name which class a parent finding gets, or say it keeps the class the parent gives, with any `decision` class mapped to `design`. Otherwise a parent finding carrying `class: decision` escalates back to the parent, which is the loop D7's rationale says it avoids.

4. **LOW, D5 (the deadline caps how long the parent can hold a round).** This holds against `review_timeout_min` (240 by default). But the deadline is counted from the ask, while the chair waits only after its own fan has finished. Neither key's validation stops a hub from setting `discussion_review_parent_timeout_min` at or above `review_timeout_min`, and then the chair's shuttle timeout kills the round before the deadline lapses. `assist_reviewer_timeout_min` is capped below `review_timeout_min` for this reason. The fix: cap the new key the same way.

5. **LOW, D5 (a new await verb).** The chair already waits on a marker with `lym shuttle await <end marker>` (`seat-directive-assistant-chair.md`). The new verb's only extra job is to enforce the deadline. That is justified, but D5 should say why the existing wait does not serve, or Plan may fold it into the shuttle wait plus a Go-side deadline writer.

6. **LOW, D6 (the fork's one write).** This is a bound the record states but nothing enforces. Orch-role's fork rule is prose, and nothing stops the fork from running `lym loom parent-review answer` for another slug, or `lym loom decision add`, which D6 also lets the orch do. It is acceptable as an instruction-level bound, but the record should say it is prose-only. The parent-only `--by` guard does not apply to the answer verb, so any session in the hub that can resolve the slug can answer as the parent. D4 should state who may run it, or that it is unguarded like the PR-Review creel answer.

**Claims attacked**

- D1, assistant keys open to `run_settings` (category `none`): held.
- D1, origin tag `assist:assist`: held (`"assist:"+AssistSeatName`).
- D2, budget 0 means the parent is never asked: held.
- D2, auto-accept, hard cap or accept can end the perch with parent findings open: held.
- D2, round 2 asks only after findings: held. The default fan runs round 2 solo, and the parent part is independent of the fan.
- D3, `NotifyParent` returns not-posted when no orch resolves, giving `no-parent`: held, per the exploration.
- D3, restart sends no second ask: broken unless keyed by round (finding 1).
- D4, run from the orch's worktree like `decision add <slug>`: held. `preRun` resolves the target from the slug through `resolveReviewTarget`.
- D4, refusal after `done`/`timeout`: open to the race in finding 2.
- D5, deadline bound: partly broken (finding 4).
- D5, keep-warm cap and wake once a second: held as a stated requirement.
- D7, never rejected: held. Class unspecified (finding 3).
- Constraint, `fable[high]` is valid: held, per the spec mapping.

**Tool calls**

- Grepped `template.yaml` for `review_timeout_min` and `decision.go` for slug resolution. My context lacked the review timeout default and how `decision add` resolves a slug from another worktree.
- Grepped the assisted reviewer stencil and the review format. My context lacked how the chair waits on an assistant and the finding-class field.
- Read `seat-directive-assistant-chair.md`. My context lacked the existing wait verb the chair uses.
