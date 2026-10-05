# Test-writing rules: tests follow behavior, not symbols

Loom tasks wrote one test per symbol and often one per refusal, which inflated the suite.
This change states the rules for writing a test and makes a redundant test a review finding.
It is stage 1 of getting the suite under control; `test-suite-prune` follows and must already write tests under these rules.

## What changes

- `PATTERN.md` gains three review-enforced Testing entries, ahead of the existing how-to entries:
  - `PATTERN-test-economy`: a test pins behavior at a module's public surface and covers something no existing test covers, else it extends an existing test; a review fix adds one only for a coverage gap.
    Its background is `pattern/PATTERN-test-economy.md`.
  - `PATTERN-test-time-seam`: time-driven behavior has an injectable interval or clock, and no test waits out a production duration.
  - `PATTERN-test-parallel`: a test calls `t.Parallel` unless it touches named process-global state.
- `PATTERN.md` also moves `PATTERN-sandbox-coverage`, `PATTERN-dev-prod-binary-separation` and `PATTERN-quarry-cgo` into a new `## Build and tooling` section, lines unchanged.
  `PATTERN-hubforge-fixtures` gains a leading condition: it applies where no fake or in-memory fixture can test the behavior.
- `PATTERN-refusal-way-forward` now requires that a test reaches each refusal row, and one table-driven test may reach many.
- `contracts/specs/loom-plan-spec.md` and `loom-template-plan.md`: a card that introduces behavior carries its coverage, extending an existing test where one covers the surface and adding a new one only for uncovered behavior.
  Card granularity keys on public surface and cross-package reuse instead of testability, and a helper serving one surface is bundled into that surface's card.
  A new `**Create:**` test file is no longer called the normal shape.
- `loom-rubric-plan-review.md` and `loom-rubric-webster-review.md` gain a test-economy bullet.
  A redundant test is a finding, and the finding must name the existing test that already covers the behavior.
- `burler-step-3-fix.md`: a fix adds a test only for a coverage gap, otherwise it corrects the existing test.
- `loom-template-discussion.md`: a must-cover scenario names a behavior, not a test to write.
- `contracts/stencils/claims_test.go`: the wording claims move to the new phrases as rows of the existing table; no test function is added.

## Design decisions

- The rule lives in one PATTERN entry because PATTERN is inlined into every spawned producer and reviewer, and into sessions outside loom.
  Stencils, rubrics and the spec point at it and restate only a finding condition or a one-clause criterion (`PATTERN-producer-pointer-rule`).
- Redundancy is BLOCKING through the existing review-fix directive.
  Requiring the finding to name the covering test bounds that: a reviewer cannot block on a vague "seems redundant".
- No mechanical redundancy detection is added; enforcement is review discipline until `test-suite-prune` builds a coverage tool.
- The PATTERN revision was committed ahead of the stencil and spec edits on this branch, at the operator's instruction, so this task's own review rounds read the new rules.
  The landed result is the same as a single commit.
- `contracts/specs/refusal-spec.md` needed no edit: its only per-row test statement is the wiring-guard row's "no per-row test".
- The planparser golden fixture and the worked example's card bodies are unchanged.
- Existing tests are not brought under the time-seam and parallelism rules here; that is `test-suite-prune`.

## To check by hand

- `scribe:testing` lives in the separate scribe repo and was not edited.
  Under `## Coverage` it needs: "A new test covers behavior no existing test covers; otherwise extend an existing test, as a table row where the test is table-driven." and "A review fix adds a test only for a coverage gap; otherwise it corrects the existing test."
  Its "Every path, not just the happy one" bullet should add that paths sharing a surface are rows of one test.
  Someone needs to carry this to the scribe repo.
- Reviewers judging redundancy without a coverage tool may over-flag; watch the first runs under the new rubrics.
- `lyx board get test-writing-rules` returned an invalid-JSON error while this description was written, so the board entry was not read and no `Closes #N` line is included; add one if the entry names an issue.
