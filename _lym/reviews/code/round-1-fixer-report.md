---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposal refuses a manual tier the key's tiers do not list. That also refuses a key listing a tier whose covers holds the manual tier, whose command runs it anyway and which ran those tests before this change, so such a key is not refused."
  - id: F2
    disposition: fixed-differently
    reason: "The proposal asserts that Slots equals the pool size. The run record stores slots: all as 0 (the gate run showed Slots:0), so the test asserts 0, against the 1 tier0 records."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The proposed sentence says marked tests run whatever tiers the key lists, which the F1 fix makes false for a manual tier, so the bullet states that exception and its refusal too."
---

# Fixer report, round 1

## Changes

- F1: `markedScope` in `internal/gatecli/only.go` refuses a marked test of a manual tier that the key neither lists
  nor covers through a listed tier.
  `internal/gatecli/doc.go` and `internal/gateslot/template.yaml` state the rule.
  `TestGateTest_Scopes` checks the refusal, a key listing the manual tier and a key listing a covering tier.
  Proposal not made: refusing every manual tier the key does not list, because `covers` names the tiers
  a tier's "command runs anyway" and `ranWhole` and `dropCovered` in `internal/gatecli/tiers.go` treat it so.
- F2: the same test reads the tier1 guard run record and asserts `GuardStep`, `Tier` 1 and `Slots` 0.
  Proposal not made: `Slots` equal to the pool size, which the record does not hold.
- F3: `_lym/landing/summary.md` no longer names `internal/gatecli/cli.go`.
  It also describes this round's changes: the manual-tier exception, the refusal text, the PATTERN bullet and the test.
- F4: the undeclared-tier refusal in `internal/gatecli/tiers.go` names guard and marked tests.
- F5: the upkeep bullet of `pattern/PATTERN-test-speed.md` states what the marked scope runs and refuses,
  the manual-tier rule included.

Commits: `fa81c9398` (F1, F2), `ddb95effd` (F4), `fb0840e6b` (F5), `1f05df819` (F1, from the self-check).
`_lym/landing/summary.md` is in no commit.

`lym gate test --tier 1 ./internal/gatecli ./internal/gateslot` passed after the last change:
266 passed, 0 failed, 110 skipped.

## Self-check

- fix-code-correctness, LOW, the manual-tier refusal ignores `covers`: fixed.
- fix-proposal, MEDIUM, the same `covers` gap: fixed, merged with the line above.
- fix-consistency, LOW, the summary's design decision still calls the key's tiers unused: fixed.
- fix-consistency, LOW, the summary's `## Changes` omits the PATTERN and `tiers.go` changes and the new test cases: fixed.
- fix-affected-set, LOW, the same stale design decision: fixed, merged.
- fix-affected-set, LOW, the same stale test bullet: fixed, merged.

All four lenses returned a report.

## Disputed

## Deferred

## Rulings
