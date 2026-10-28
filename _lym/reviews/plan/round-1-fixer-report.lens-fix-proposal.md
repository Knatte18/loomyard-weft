```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the review's proposals that the fixer carried out as written: F1, F2, F3, F4, F5 (option 1), F6, F9, F10, F11 and F12.

**[MEDIUM] F3 dropped done-card rows that pin something new**
- **Changed text:** `_lym/plan/08-seam-test-check.md` and `_lym/plan/09-doc-go-check.md`, Tests.
  The done-card rows were removed from the D6 and D8 rows of `TestValidatePlan_CardChecks`.
- **Summary:** card 7's done-card row does not cover `seamTestFindings` or `docGoFindings`.
  Each of them receives both `plan` and `cards`, so an implementation that walks `plan.Cards` would check done cards and still pass every remaining row.
- **Evidence:**
  - Card 8 declares `seamTestFindings(plan *planparser.Plan, cards []planparser.Card, …)`, and card 9 declares `docGoFindings(plan *planparser.Plan, cards []planparser.Card, …)`.
  - In `internal/loomshed/gates.go:259-269`, `ValidatePlan` builds `pending` and calls `planContextFindings(plan, pending, …)`.
    Skipping done cards therefore depends on each check iterating `cards`, which card 7's row proves for `testTierFindings` alone.
  - `ValidateReworkPlan` (`gates.go:330`) passes `plan.Cards`, so no rework row tells the two apart either.
  - The review's premise, "every mode reaches planContextFindings through the same pc and pending filter", holds for the call.
    It does not hold for each check's own iteration.
- **Way out:** restore the done-card row in cards 8 and 9.
  The `ValidateDispatch` drops and the loomcli mode-row drops stand.

**[LOW] F6's new step does not fit every reader error**
- **Changed text:** card 8 Intent and Docs.
  The new sentence says "the operator moves or corrects the marker in the task worktree as card 5's way forward says".
- **Summary:** the reader returns any `*GuardScanError` from `FindSeamTests`.
  That includes the `GuardParse` error for an unparsable test file, which card 5 says `guardNamesInFile` already returns.
  That error has no marker to move and no card-5 way forward, so the named step cannot be taken for it.
- **Evidence:**
  - Card 5 Intent: "an unparsable file is the `GuardParse` error `guardNamesInFile` already returns".
  - Card 8: "its error, a `*GuardScanError` included, is returned".

No finding for the others:
- **F1:** skipping a non-`.go` target follows D8's "non-test Go code".
- **F2:** the extended way forward reaches a pass in every failing state the card pins.
- **F4:** `validate-plan` accepts the file target with only an informational `create-new-unit`, and card 4's Intent states its tier line.
- **F5 option 1:** asserting `glyph-not-found` is consistent with the quarry output the fixer showed.
- **F9:** a token with an argument list holding whitespace is skipped before classification, so only whitespace-free lists reach the strip.
- **F10:** naming the consumers as a group holds.
- **F11:** naming `PlanIndex` in the `wireLightweight` comment holds.
- **F12:** every caller still guards on a non-empty list, and without the branch an empty call is still harmless.

Tool calls:
- Read `internal/loomshed/gates.go:255-275` and `:318-335` to check whether each plan-context check, rather than the shared call, applies the pending filter. My context held only the review's citation of those lines.
