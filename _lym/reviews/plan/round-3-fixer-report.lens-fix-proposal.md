```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed (F1, F2, F4, plus the parent's set-of-tags decision) by building a case where the proposal is wrong.

No findings.

- **F1, the support-only `untagged` clause in the way forward:** I checked whether naming "helpers" alongside `TestMain` wrongly steers a shared helper to `untagged` when it needs a `||` disjunction. It doesn't. The clause mirrors `pattern/PATTERN-test-speed.md:17` ("an untagged test file holds support only, such as `TestMain` and helpers"). The existing `||` clause mirrors line 22 (a helper used by two tiers carries the disjunction). Both forms pass the check, so neither reading leaves a writer stuck.
- **F2, the row for an untagged helper beside `tier0`:** On one line, `helper_test.go` holds the word `untagged` plus `tier0`, and `x_test.go` holds `tier0` plus the word `untagged`. Both files fail whether the word reaches them through the carry rule or by sitting on the line. Split over two lines, each passes, and base-name matching does not cross between `x_test.go` and `helper_test.go`, since neither name is the other's tail. PATTERN line 22 puts a one-tier helper inside that tier's file, but this row only exercises the check, and whether the tier fits is a stated bound of the check. So that is no defect.
- **F4, the Long help sentence owned by card 7 and extended by cards 8 and 9:** The plan's cards apply in number order, and cards 8 and 9 add their names outside the "which read" clause, so neither overwrites card 7's rewrite. Placing the new checks in "every mode but --require-approved" agrees with card 7 (rework runs `planContextFindings` over the fresh generation) and with card 8 (both callers of `planContextFindings` pass the index).
- **Parent decision, tags as a set:** The two-line `tier1` row has no line naming another test file, so the carry rule does not reach it and it passes under set collection.

Further tool calls:
- grep of `pattern/PATTERN-test-speed.md` for untagged, helper and `TestMain`: my context did not hold PATTERN's wording on what an untagged test file and a shared helper may hold.
