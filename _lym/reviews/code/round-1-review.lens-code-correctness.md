```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each rule the diff adds or changes against concrete splits.

**Rules and the cases I ran**

- **Rubric "The cut runs the most parts at once", first sentence** (`contracts/stencils/loom/loom-rubric-raddle-review.md:35`): a split with 3 parts, where the work gives 3, plus one extra part that only makes two others run side by side. The rubric flags the extra part. Correct.
- **Same bullet, cut choice** (`:36-38`): two 4-part cuts with the same chain length of 2.
  - Cut X is P1 then {P2, P3, P4}: 3 parts run at once.
  - Cut Y is P1→P2 beside P3→P4: 2 parts run at once.
  - "The one with the shortest `depends_on` chain" treats X and Y as equal, though X runs more parts at once. The bullet's heading is about how many parts run at once; its test only measures chain length. Not correct (LOW, finding 2).
- **Same bullet, finding condition** (`:38`): a 3-part cut A→B→C where the work forces the chain, and no other 3-part cut is shorter. No finding. Correct.
- **Rubric "No part guesses at another's interface"** (`:39-41`). Correct in all three cases:
  - A builds X, B uses X, no `depends_on`, interface not fixed: a finding.
  - Same, with X's interface fixed in both bodies: no finding.
  - B depends on C, which depends on A: chained through a path, no finding.
- **Rubric `design` class sentence** (`:66-68`): the A→B→C cut above. Read literally, "a cut that runs fewer parts at once than its part count allows" flags it, because 3 parts allow 3 at once and this cut runs 1. The judgement bullet at `:38` only flags a chain that another cut with the same part count shortens. Not correct (MEDIUM, finding 1).
  - The earlier cases are still there: a decision no part can build, a seam that does not hold, a recipe or profile that does not fit.
- **Agent stencil rule one** (`contracts/stencils/raddle/raddle-template-agent.md:49`): an agent tempted to split one package's change in two so the halves run side by side. The rule forbids it. Correct.
- **Agent stencil rule two** (`:50-51`): two unordered parts that both list `internal/x`. The rule and the existing bullet at `:48` both forbid it, and the `packages-shared` gate does too (`internal/raddleparser/check.go:398`). Correct.
- **Agent stencil rule three** (`:52-54`): the same cases as the rubric's interface bullet. The outcomes match the rubric's. Correct.

**Findings**

1. MEDIUM, design. `contracts/stencils/loom/loom-rubric-raddle-review.md:68`: the `design` class says "a cut that runs fewer parts at once than its part count allows". Read literally, that makes every chain a design fault, even a chain the work forces. The judgement at `:38` only faults a chain that a cut with the same part count shortens. The class sentence should say "fewer parts at once than another cut with the same part count".
   - Evidence: `loom-rubric-raddle-review.md:38` and `:68`.
2. LOW, design. `contracts/stencils/loom/loom-rubric-raddle-review.md:36-37`: "the one that lets the most parts run at once" is defined as "the one with the shortest `depends_on` chain". Two cuts with equal chain length can differ in how many parts run at once (cut X against cut Y above), so the test the bullet gives does not measure what its heading names.
   - Evidence: `loom-rubric-raddle-review.md:34-38`. The record's own wording is "a short `depends_on` chain, and parts with no dependency between them touching different packages" (`_lym/discussion/decision-record.md:22`).

Tool calls: none.
