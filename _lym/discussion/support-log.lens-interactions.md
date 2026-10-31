```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of the record's decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches. Two pairs break in a way that stops or bypasses the landing.

**Findings**

1. **BLOCKING** — D1 × D2 × D3: Describe can be skipped and leave the run with no change description.
   - Describe keeps darn's `entry_gate: budget.Code-Review`. That gate holds only while the Code-Review soft cap is non-zero (`internal/shedbuild/doc.go:30`: "A bare `budget.<Segment>` operand holds when it is non-zero").
   - In darn this works because the writer has already written a description (`contracts/recipes/darn-recipe.yaml:39`, `:57-58`: "Describe's entry gate skips it when the perch is off"). D8 gives the sampler agent no description to write.
   - D2 copies darn's `speed` block, `code_max_bounces: 0`, into a sampler block. A sampler task on `speed`, or any hub or task whose `loom.code_max_bounces` is 0, therefore skips Describe.
   - Finalize, which parses the change description first, then has none to read.
   - D3's switch and the budget are two separate off switches, and only the budget skips Describe.
   - Fix: Describe on sampler carries no `entry_gate`, or the agent writes a description as darn's writer does.

2. **BLOCKING / MEDIUM** — D1 × D3: Finalize's reroute ignores `sampler.code_review`.
   - Finalize decides whether a review follows its reroute through `rerouteReviewed` and `codeBudgetZero` (`internal/loomcli/landingdeps.go:390-404`). Both read only loom's code budget, never `sampler.code_review`.
   - With `code_review` false and the hub's default budget above 0, Finalize believes a review follows. A spent-cap or over-cap fixer change then reroutes to `Code-Bouncer`.
   - `Code-Bouncer` is Skipped by `entry_gate: sampler.code_review`. The run goes on through Describe and PR-Review back to Finalize, so the fixer's change lands with no review and no halt, or circles.
   - `CodeReviewOff` misreads the same way.
   - D3's switch and the landing's budget reading are two mechanisms over the same state, and they disagree.
   - Fix: make the budget the single off switch, or have `codeBudgetZero` and `rerouteReviewed` read the switch on a sampler run.

3. **MEDIUM** — D6, pause × dead strand: the row can halt on a session it stopped itself.
   - D6 stops the session at a covered wake and resumes it at the first uncovered wake. It also halts whenever a wake finds the recorded strand dead.
   - The order of these two checks at the first uncovered wake is not stated, and neither is how a strand the row stopped is told apart from one that died.
   - If the liveness check runs first, every pause on a sampler run ends in Stuck with Halt.
   - Fix: state that the resume runs before the liveness check, and that a strand the row stopped is not "dead".

4. **LOW** — D4 × D5 and D6: the admitted strand after a halt-and-resume is not stated.
   - D4 admits "the strand the row recorded". After a dead-strand halt, `lym loom resume` resumes the recorded session id.
   - The record does not say whether the resumed strand's name, which the caller rule matches, is re-recorded.
   - If it is not, `decision add` from the resumed agent is refused.

5. **LOW** — D6 × D4: the dirty-tree refusal points at the wrong commit.
   - If `decision add`'s weft commit fails, `_lym` is left dirty.
   - `sampler-done` refuses a dirty tree "naming a commit". The right step there is `lym fabric commit`, which the record does not name.

6. **LOW** — D2 × D3: the copied perch values do nothing by default.
   - The ladder profiles' sampler blocks copy darn's perch rounds but not `sampler.code_review`.
   - A task naming `solid` therefore still has no review unless `run_settings` turns it on. The values do nothing unless the task also turns the perch on (on `speed` they are what triggers finding 1).
   - This is consistent with "off by default", but the record should state it, or the blocks should carry `code_review`.

**Pairs checked**

- D1 Describe gate × D3 switch × D2 `speed` block: broken (finding 1).
- D1 Finalize reroute × D3 switch: broken (finding 2).
- D6 pause stop/resume × D6 dead-strand halt: broken (finding 3).
- D4 caller strand × D5 resume and D6 halt-resume: ambiguous (finding 4).
- D4 decision add commit × D6 dirty-tree check: held, but the way forward is wrong (finding 5).
- D2 profile blocks × D3 switch: held, values inert (finding 6).
- D5 unawaited session × D7 waiting-row list: held; dobby's quiet notice is off while at the row, and the session is stopped at Done before the child leaves the list.
- D6 Done stops session × D4 "after row ended, refused": held.
- D6 entry stamp × `goto` and resume: held.
- D6 flag × D6 record-damage check: held.
- D7 watchdog exemption × D6 halts: held; the halt ends the step, and the list no longer applies.
- D8 notes-only board writes × the board's run-agent rule: held.
- D9 own `tmux -L` server × D6 `sampler-done`: held, stencil-only, as its bound states.
- D1 `verify_source: tree_scope` × D3 module: held; it reads darn's tree scope, as stated.

**Acceptance criteria and constraints**

- The second D1/D3 criterion (perch Skipped when off) does not cover Describe or Finalize under a zero or non-zero budget, so findings 1 and 2 go untested.
- The D1/D6 empty-landing criterion depends on finding 1 when the budget is 0.
- `PATTERN-shed-recipe-registry` (a row left out only by its `entry_gate`) holds for the perch rows. It is undermined because the landing reads a different off signal (finding 2).

**Tool calls**

- Grepped the shed recipe spec, the entry-gate code and the Finalize reroute: the record cites neither `RerouteReviewed` nor the meaning of `budget.` gates.
- Grepped how `RerouteReviewed` and `CodeReviewOff` are computed in `internal/loomcli/landingdeps.go`: needed to test D1 × D3 and not in context.
- Grepped the `budget.<Segment>` gate meaning and darn's Describe comments: needed to test D1 × D2, and the record carries the gate without its meaning.
