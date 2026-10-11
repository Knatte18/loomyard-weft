# Card 6 — parent-review-last-gate

**Edit:**
- `internal/shedrecipe#resolveRowGateSpec`
- `internal/shedrecipe#resolveGateSpec`
- `plan:internal/shedrecipe#TestResolveGateSpec_ParentReviewElementRefusals`

**Prosa:**
- `contracts/specs/shed-recipe-spec.md`
- `contracts/specs/refusal-spec.md`

**Intent:**
D7: `parent-review` must be the last entry of its gate list, since a must-pass entry after a pending `parent-review` would read `not_reached` at a deadline finalize while the review is pending.

`resolveRowGateSpec` refuses a list in which a resolved `parent-review` element is followed by another element:
`shedrecipe: <entry>: config key "gates" element <i>: "parent-review" must be the last element of the list; way forward: move the parent-review element to the end of the gates list`, with `<i>` the `parent-review` element's index.
The input that reaches it, after every element resolves, is a row whose `gates` list names `parent-review` and then any other valid gate, which a developer editing the recipe writes.
`resolveGateSpec`'s doc states the rule beside the one-name-per-list rule.

`TestResolveGateSpec_ParentReviewElementRefusals`, over `parentReviewEnv`, gains two rows:
`parent-review` then `discussion` refuses naming the row, `"parent-review" must be the last element` and `way forward:`;
`discussion` then `parent-review` builds two entries with `parent-review` last.

`shed-recipe-spec.md`'s gate paragraph adds that `parent-review` must be the last element of its list, else construction fails.
`refusal-spec.md` gains a loom-section row, `recipe: parent-review not last`: a recipe's `gates` list names another element after `parent-review` when the row builds; correctness halt; move the `parent-review` element to the end of the `gates` list.

**ImpactSummary:** one new build-time refusal in the shared gate resolver; the embedded recipe already lists parent-review last, so no shipped row changes.
