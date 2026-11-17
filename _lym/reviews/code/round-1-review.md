---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/raddleparser/check.go:431
    summary: The proposal check accepts a part whose labels list one module label twice, so the board refuses that part only at Raddle-File, after the proposal is approved.
    fix: >-
      In `vocabulary()` in internal/raddleparser/check.go, also report a part label that the part's `labels` list more than once.
      Report it once per repeated label, under a new check constant `CheckLabelDuplicate = "label-duplicate"` declared beside `CheckLabelUnknown`.
      Give its Detail a way-forward clause such as "way forward: list each module label once in labels".
      Extend the `vocabulary` doc comment to cover the repeat.
      Add a `label-duplicate` bullet after `label-unknown` in the check list of contracts/specs/raddle-proposal-spec.md: a part that lists one module label more than once.
      Add a `TestCheck` row in internal/raddleparser/check_test.go that sets `p.Parts[0].Labels = []string{"mod-a", "mod-a"}` and wants `[]string{CheckLabelDuplicate}`.
---

### [MEDIUM] A repeated part label passes the check and fails at filing

**Location:** `internal/raddleparser/check.go:431`

**Issue:** `vocabulary()` tests each part label only for membership in `LabelVocabulary`, so `labels: ["mod-a", "mod-a"]` produces no finding.
`raddle-propose`, the gate and `raddle-approve` all pass such a proposal.
Raddle-File then files the part with `append([]string{part.Type}, part.Labels...)` (`internal/loomshed/raddlefile.go:507`), which keeps the repeat.
The board write's `validateLabels` refuses an entry that carries a label more than once (`internal/boardengine/labels.go:28-30`).
An approved proposal therefore halts the run at the board write instead of being refused at the proposal check, where the agent could correct it from a way forward.
Before this diff every filed label came from the board entry itself, which the board had already validated, so the repeat is new with the per-part `labels` key.

## Rejected

- code-tests, LOW consistency at `internal/raddleparser/check.go:431-438`, saying no test asserts the new findings' Detail way-forward text.
  Rejected because `TestCheck` already pins that each refusal fires (check ids) under its own row.
  The rule requiring a test to assert a refusal's message, `PATTERN-refusal-way-forward`, binds only the `webster*`, `shed*` and `loom*` packages, and `raddleparser` is none of them.
  Asserting the Detail wording would be an assertion on static prose, which the rubric's test-economy bullet itself counts as a finding.

The handler's holistic review found the diff matches every bullet of the board entry's Idea:
- the proposal names per-part labels and Raddle-File files them;
- the check refuses a missing or out-of-vocabulary label and accepts one outside the Sketch's set;
- the split perch's rubric weighs labels against packages;
- the stencil and spec state the rule.

The change description's claims hold against the diff.
No commit in the range carries a `Pattern-Departure:` trailer, and the directive declares no PATTERN changes.
