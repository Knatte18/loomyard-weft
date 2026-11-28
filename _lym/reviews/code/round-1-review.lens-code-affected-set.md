```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changes force outside the lines it touches.

**List, per change**

1. **First-caller rule** (`loom-plan-spec.md:366-368`, `loom-block-plan-cards.md:56-58`, `loom-rubric-plan-review.md:58-62`)
   - `pattern/PATTERN-test-only-code.md:30-32` says the guard carries `//lym:only upkeep`, no task gate runs it, and "A card may therefore add a function that only a later card calls." The diff contradicts it. **Not covered.**
   - `cmd/lym/testonlycode_linux_test.go:91` carries `//lym:only upkeep`, and lines 48 and 66-69 use the allowlist for functions whose first caller comes in a later part. The diff's premise ("can fail on any card between the two") is false against this code. **Not covered.**
   - Commit `2326ef4da`, an ancestor of base `5c985d926`, removed these same rules from the spec, `loom-block-plan-cards.md` and `loom-rubric-plan-review.md` on purpose ("No card gate runs that guard now"). The diff puts them back almost word for word. **Not covered.**
   - `loom-plan-checks-spec.md`: the check is gone and the diff adds none. Consistent.
   - Includers of `loom-block-plan-cards` (`loom-template-plan.md:83`, `loom-template-part-writer.md:59`, `loom-template-parts-chair.md:73`) receive the rule. Covered.
   - Granularity bullet `loom-rubric-plan-review.md:54-57` still says a public-surface symbol "gets its own card even when one card is its only consumer". It does not call the new bullet an exception, though the spec does. Partly covered.
   - Commit message of `e489adf46` and `_lym/landing/summary.md` repeat the false premise. **Not covered.**
2. **Fact checks on a `fix`** (`burler-review-format.md:33-40`)
   - All review paths render the format: solo, cluster fan, assist fan, assisted chair and fixer via `review_format_path` (`prompt.go:57,70,88,111,167`). Covered.
   - Lens forks and assistant boilerplate return findings with no `fix`, and the handler writes the fix. Covered.
   - `burler-parent-consolidation.md:20` says a kept parent finding's `fix` "carries the item's `change`", and line 23 says it is never rejected. The new "Never write a `fix` that rests on a fact the code disproves" gives no route for a parent change the code disproves. **Not covered.**
   - Fix-self-check forks and `burler-fix-check-rubric.md` write no `fix` key. Not affected.
   - Plan-Review "Proposed fixes" (`loom-rubric-plan-review.md:150-166`) works alongside the new rule. Covered.
   - `internal/burlerengine/template_test.go:142-146` checks markers only, and the diff adds no marker. Covered.
3. **New `decision` route** (`burler-review-format.md:39`)
   - `shed-recipe-spec.md:117-134` escalates `decision` findings when `escalate_decisions` is set, and `decisions_escalated` keeps them from the fixer. This is consistent with the format's class meaning at line 47. Covered.
   - `burler-parent-consolidation.md:19` maps a parent `decision` to `design`. That is unchanged, but it interacts with the gap in item 2.
4. **Fixer step reads unverified facts** (`burler-step-fix.md:29-30`)
   - It agrees with the dispositions at lines 34-50: "own change" is disposition 2, and "defer" is the deferred route. Covered.

**Findings**

- **F1. BLOCKING, design.**
  - Location: `contracts/stencils/loom/loom-block-plan-cards.md:56` (also `loom-plan-spec.md:366`, `loom-rubric-plan-review.md:58`).
  - Summary: the first-caller rule rests on a premise the repository disproves and contradicts `PATTERN-test-only-code`'s background. The reachability guard carries `//lym:only upkeep`, no card gate runs it, and the background says "A card may therefore add a function that only a later card calls." Commit `2326ef4da` removed these same rules on purpose.
  - Evidence: `pattern/PATTERN-test-only-code.md:30-32`; `cmd/lym/testonlycode_linux_test.go:90-92`; `git show 2326ef4da` (body: "No card gate runs that guard now", and the removed lines in the same three files).
- **F2. MEDIUM, consistency.**
  - Location: commit `e489adf46` message and `_lym/landing/summary.md` (`## What changes` and `## Design decisions`).
  - Summary: both claim a reachability check "can fail on any card between the two". The code disproves this (`//lym:only upkeep`).
  - Evidence: `cmd/lym/testonlycode_linux_test.go:91`; `pattern/PATTERN-test-only-code.md:30-32`.
- **F3. MEDIUM, consistency.**
  - Location: `contracts/stencils/burler/burler-review-format.md:37`.
  - Summary: "Never write a `fix` that rests on a fact the code disproves" conflicts with parent consolidation. There the kept parent finding's `fix` must carry the parent item's `change` and can never be rejected, and the diff gives no rule for a parent change the code disproves.
  - Evidence: `contracts/stencils/burler/burler-parent-consolidation.md:20,23`.
- **F4. LOW, consistency.**
  - Location: `contracts/stencils/loom/loom-rubric-plan-review.md:57`.
  - Summary: the granularity bullet still says a public-surface symbol gets its own card "even when one card is its only consumer". It does not mark the new first-caller bullet as its exception, though the spec (`loom-plan-spec.md:366`) and the writer block (`loom-block-plan-cards.md:56`, "even when it is part of a module's public surface") do. This finding becomes moot if F1 drops the rule.
  - Evidence: `loom-rubric-plan-review.md:54-62`.

**Tool calls**

1. grep for `create-caller-later`, first-caller wording and prose-asserting tests: to find rule documents and tests the manifest lacked.
2. grep of `markers_test.go` and `template_test.go`, plus `git show 2326ef4da` on the three plan files: to see what the tests assert and what the earlier commit removed.
3. Read of `internal/burlerengine/template_test.go:125-164`: to confirm the review-format test checks markers only.
