```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. The diff is prose only, so I traced each new rule as a branch: I fed it one concrete case and followed it through the rules it meets.

**Per branch: the case I ran, and whether it is correct**

1. `burler-review-format.md:33-36`, the fact checks before a `fix`.
   - Case: a fix reuses an unexported identifier from an external `_test` package (#288). The package-clause and export check catches it. Correct.
   - Case: "this test only reads", where a callee takes a lock that creates a file. The callee-chain check catches it. Correct.
   - Case: a fix names a recorded-session kit that does not exist (#285). The existence check catches it. Correct.
2. `burler-review-format.md:37-39`, a fact the code disproves.
   - Case: a new finding where a fix holds without the fact. The reviewer proposes it and names the gap. Correct.
   - Case: a new finding where no such fix exists. It becomes `decision`, and `shed-recipe-spec.md:118,123-124` routes it to escalation and keeps it from the fixer. Correct.
   - Case: an `open` prior `design` finding carried forward whose fix the reviewer now finds disproved. `burler-review-format.md:83` says "A carried finding keeps its `class`", so the reviewer cannot take the new `decision` route. Incorrect (F3).
3. `burler-review-format.md:40`, an unverified fact.
   - Case: the reviewer could not read a callee. The `fix` names it as unverified. Correct.
4. `burler-step-fix.md:29-30`, the fixer's handling of an unverified fact.
   - Case: the fact the code disproves is the finding's own premise. The #288 `SharedHub` case is one: the finding's premise is that the test only reads. The new line allows only "fix with your own change or defer". The three dispositions that follow put Dispute (`premise false`) first. Incorrect (F2).
   - Case: the fact only backs the proposed change. Fix differently. Correct.
5. First-caller rule, case A: function `F` is created on card 1, a test on card 2 calls it, and its first production caller is on card 4.
   - The rule, together with "A test-only caller does not count", moves `F` to card 4.
   - Card 2's test then uses a later card's target. That is refused by `uses-later-target` and breaks Plan vs. schedule (`loom-plan-spec.md:41-45`).
   - The rubric's fix "moves the function onto the card that adds its first production caller" (`loom-rubric-plan-review.md:62`) yields a plan the gate refuses. Incorrect (F4).
6. First-caller rule, case B: the first production caller is itself unreachable from a `main` until a later card.
   - Co-locating the function with that caller does not make it reachable, so the stated rationale ("can fail on any card between the two") is not answered. Incomplete (part of F1).
7. First-caller rule against the actual gate.
   - `cmd/lym/testonlycode_linux_test.go:91` carries `//lym:only upkeep`, so no card gate runs the reachability guard.
   - `pattern/PATTERN-test-only-code.md:30-32` says "A card may therefore add a function that only a later card calls."
   - Commit `2326ef4da`, an ancestor of base `5c985d926`, removed the `create-caller-later` check and its stencil rules for exactly this reason.
   - The rule's premise is false. Incorrect (F1).
8. Card claims and cross-card checks (`loom-block-plan-cards.md:60-64`, `loom-rubric-plan-review.md:63-71`).
   - Case: each of the five #291 instances. Seam reachability, a reader through the call chain, the `cmd/lym` detached-child read (call chain), a cross-card contradiction on `review` mode, and the input reaching a branch past an earlier check are each covered by a clause. Correct.

**Findings**

- **F1, BLOCKING, design.** Location: `contracts/specs/loom-plan-spec.md:366-368`, `contracts/stencils/loom/loom-block-plan-cards.md:56-58`, `contracts/stencils/loom/loom-rubric-plan-review.md:58-62`.
  - Summary: the first-caller rule rests on a false premise. The reachability guard runs only in upkeep, never in a card gate. The rule contradicts the `PATTERN-test-only-code` background, which explicitly allows a card to add a function that only a later card calls, and it re-adds the rule that commit `2326ef4da` deliberately removed.
  - Evidence: `cmd/lym/testonlycode_linux_test.go:91`, `pattern/PATTERN-test-only-code.md:30-32`, `git show 2326ef4da` ("Plan check removed").
- **F2, MEDIUM, consistency.** Location: `contracts/stencils/burler/burler-step-fix.md:29-30`.
  - Summary: the unverified-fact line limits a disproved fact to "fix with your own change or defer". When the disproved fact is the finding's premise, Dispute `premise false` is the disposition that the ordered list beside it (lines 31-41) requires first, and this was the actual #288 outcome.
  - Evidence: `burler-step-fix.md:31-36`; decision record, #288 bullet 1.
- **F3, LOW, consistency.** Location: `contracts/stencils/burler/burler-review-format.md:37-39`.
  - Summary: the rule "raise the finding as class `decision`" cannot apply to a carried finding, because a carried finding keeps its class (line 83). The text does not say what a carried non-`decision` finding with a disproved fix does.
  - Evidence: `burler-review-format.md:81-83`.
- **F4, MEDIUM, design.** Location: `contracts/stencils/loom/loom-rubric-plan-review.md:62`, `contracts/stencils/loom/loom-block-plan-cards.md:56-58`.
  - Summary: moving a function onto its first production caller's card, with test-only callers ignored, makes an earlier card's test use a later card's target. The resulting plan is refused by `uses-later-target` and violates Plan vs. schedule. The rule does not say how test callers on cards in between move.
  - Evidence: `loom-plan-spec.md:41-45`; `loom-rubric-plan-review.md:152`, which says a fix proposes only what every check admits.

No tool calls made.
