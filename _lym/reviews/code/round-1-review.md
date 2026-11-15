---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/loom/loom-rubric-raddle-review.md:68
    summary: The design class names a cut that runs fewer parts at once than its part count allows, which faults every chain, even one the work forces, and omits the part added only for concurrency.
    fix: In contracts/stencils/loom/loom-rubric-raddle-review.md, in the design class sentence at lines 66-68, replace "a cut that runs fewer parts at once than its part count allows" with "a part added only so parts run at once, a cut that runs fewer parts at once than another cut with the same part count". Leave the sentence's other cases unchanged.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/loom/loom-rubric-raddle-review.md:36-38
    summary: The judgement defines the cut that runs the most parts at once as the one with the shortest depends_on chain, so two cuts with equal chain length but different numbers of parallel parts count as equal.
    fix: In contracts/stencils/loom/loom-rubric-raddle-review.md, replace line 37 "the one with the shortest `depends_on` chain." with "a short `depends_on` chain, and the most parts with no `depends_on` path between them." and replace line 38's ending "and so is a chain a cut with the same part count shortens." with "and so is a cut that runs fewer parts at once than another cut with the same part count."
---

### [MEDIUM] The design class faults every chain and leaves out the added part

**Location:** `contracts/stencils/loom/loom-rubric-raddle-review.md:68`

**Issue:** The rewritten `design` sentence names "a cut that runs fewer parts at once than its part count allows".
Read literally, a 3-part cut A→B→C runs one part at once, while its part count allows three, so any chain is a `design` finding, even a chain the work forces.
The judgement bullet at line 38 faults only a chain that a cut with the same part count shortens, so the two disagree.
The sentence also leaves out the other case the judgement makes a finding: a part added only so parts run at once.
Every earlier case is kept: a decision no part can build, a seam that does not hold, and a recipe or profile that does not fit its part.

### [LOW] The concurrency judgement measures only chain length

**Location:** `contracts/stencils/loom/loom-rubric-raddle-review.md:36-38`

**Issue:** The judgement is headed "the most parts at once", but its test is "the one with the shortest `depends_on` chain".
Two 4-part cuts with a chain length of 2 can differ: P1 then {P2, P3, P4} runs three parts at once, and P1→P2 beside P3→P4 runs two.
The record's rule names both means: "a short `depends_on` chain, and parts with no dependency between them touching different packages" (`_lym/discussion/decision-record.md:22`).
The different-packages half is rightly left to the `raddle` gate's `packages-shared` check (`internal/raddleparser/check.go:398`), as the rubric's "Already enforced" section requires.
But the count of parts with no path between them is not a gate check, and the bullet drops it.

## Rejected

- Lens code-affected-set, NIT: the agent stencil's new rule-two bullet repeats the existing "keep two parts with no order between them off the same package". The board entry asks for this wording, the two bullets agree, and removing either half drops the record's wording or rewrites a rule the record did not touch.

## Focus verdicts

- The different-packages half: the rubric bullet leaves it out, and that is no defect. The `packages-shared` gate check fails any two unordered parts that share a package, and the rubric tells the reviewer never to re-derive gate checks. The chain-only test is a separate gap, filed as F2.
- Rule one: both files say that the part count stays what the work gives, and the rubric makes a part added only for concurrency a finding (line 38). The design class omits it, which F1 covers.
- Rule three: the stencil's two outcomes, chain with `depends_on` or fix the interface in both bodies, match the rubric's, and the rubric's finding condition is the stencil's prohibition. No defect.
- The agent stencil: the new bullets do not contradict the different-packages bullet or the fewer, larger parts bullet. "Never add a part" limits only adding parts, so "smaller darns stay" holds.
- The `design` sentence keeps every earlier case. Its new case is wrong, which F1 covers.
- Scope: the diff changes only the rule the record asks for. The proposal spec, the docs and PATTERN need no change, and none is declared.
- Test economy: the diff is static prose, which is exempt. `contracts/stencils/rubric_test.go:53` still pins the rubric's markers.
