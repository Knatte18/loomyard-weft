```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over diff `3a854635..HEAD`, which changes YAML, the `code-scope` lens procedure and test expectations, with no Go function changed.

**List: each changed function or branch, the case I ran, and the result**

1. **Template `review-code` fan.** `internal/lenses/template.yaml:419-423`, with no `lens_set` and no hub file.
   `ResolveFan` returns `code-correctness, code-affected-set, code-tests, code-scope`.
   All four are defined at `template.yaml:282-323`, and 4 is far below `MaxFanLenses` = 16 (`config.go:47`).
   **Correct.**
2. **csharp layer `review-code` fan.** `internal/lenses/sets/csharp.yaml:49-54`, under `lens_set: csharp`.
   The fan resolves to the four base entries in base order, then `numerics` (`csharp.yaml:44`).
   That meets the base-first rule in `doc.go` ("A layer fan holds the base fan's entries first, in base order").
   5 is at most 16. **Correct.**
   Boundary: before this diff, the csharp list (3 base entries + `numerics`) would not have been a prefix superset of a 4-entry base fan. Adding `code-scope` was therefore required. **Correct.**
3. **Emptied-fan subtest.** `config_test.go:717`.
   Input `remove: [code-correctness, code-affected-set, code-tests, code-scope]` takes all four entries out of `review-code`.
   Those names appear in no other template fan (`template.yaml:395-427`), so only `review-code` empties.
   `applyRemove` moves it to `EmptiedFans`, and `ResolveFan` refuses it naming `code-correctness`.
   With the old 3-name list, `code-scope` would stay and the `held` assertion at `:727` would fail, so the edit is the needed one. **Correct.**
4. **Count and order expectations.** `config_test.go:359, 374, 392`.
   There are 38 lenses after one is added, and `review-code` has 4 entries in template order.
   `review-code` has the `review-` prefix, so `code-scope` is put in `procedureLenses`.
   Its prose ends with "Your list: … or none." and has no emphasis line, which matches the procedure-lens shape.
   `writerFanGaps` has no writer fan mapped for `review-code` (`:404`), so no gap is raised. **Correct.**
5. **Phrase assertion.** `config_test.go:441-444`, run against the field-joined lens text.
   "A change that traces to no decision is unrequested scope, a finding of class `scope`" joins across the line break at `template.yaml:316-317` into one contiguous phrase.
   "A decision no change builds is a finding of class `scope` too." also joins across `:318-319`. **Correct.**
6. **The `code-scope` procedure on a plan-backed record.**
   Input: a record with `### D1 — …`, `### D2 — …` under `## Decisions` and one `### Added after Discussion (…)` entry.
   The lens lists D1, D2 and the added entry. That matches `DecisionIDs` and `walkDecisionHeadings` (`decisionids.go:96-131`), which take `D&lt;n&gt;` from conforming H3s and exempt added headings. **Correct.**
7. **The `code-scope` procedure on a record with no ids.**
   Input: a record whose `## Decisions` is empty, as in this run's record.
   The lens falls back to "each goal point of the amended board entry". The word "amended" brings the `Added after Discussion` entries in. **Correct.**
8. **The `code-scope` procedure on a record-alone run whose writer filled `## Decisions`.**
   Input: a darn record whose `## Goal` is the board body, while the writer filled `## Decisions` with `### D1 — …` and `### D2 — …`, one of which names work the board entry never asked for.
   The lens lists D1 and D2 as the decisions, because the fallback fires only when there is "no decision id".
   A change building the writer's own D2 then traces to a decision and is not flagged.
   The rubric says that, with a decision record alone, the other sections "are the writer's own claim … never use them as the measuring stick".
   **Wrong result.**

**Findings**

- **MEDIUM, design.** `internal/lenses/template.yaml:310-313`, the `code-scope` decision list.
  - **Summary:** in a record-alone (darn) run, the lens takes the writer's own `### D&lt;n&gt;` headings as the decisions to trace against.
    A writer can therefore license its own unrequested scope by recording it as a decision.
    The fallback to the amended board entry's goal points applies only when the record has no id.
  - **Evidence:**
    - `template.yaml:310-313`: "List the record's decisions: each `### D&lt;n&gt; — &lt;title&gt;` heading … a record with no decision id lists each goal point of the amended board entry instead."
    - The rubric section "The answer key takes one of two forms" (`contracts/stencils/loom/loom-rubric-code-review.md`, rendered in instruction-1-explore.md:117-119): with a decision record alone, "The record's other sections are the writer's own claim, made before coding and reviewed by no one: … never use them as the measuring stick."
  - **Fix direction:** key the decision list on the answer key's form, not on whether ids are present. With a decision record alone, list the amended board entry's goal points and the `Added after Discussion` entries. Use the `### D&lt;n&gt;` headings only when a plan is the answer key.

No other function, branch or error path in the diff gave a wrong result.

Tool calls: none. The inherited context held the diff, the template and csharp spans, the test spans, `decisionids.go`, `config.go`'s `MaxFanLenses` and the rubric text.
