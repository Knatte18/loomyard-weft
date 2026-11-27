---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:21
    summary: The Remaining gap section says the repository-wide-intent plan rule covers any card that changes command help in a package it does not name, but the rule binds only a card whose Intent covers every instance of one kind across the repository.
    fix: "In _lym/landing/summary.md, replace line 21 ('The plan rule `a69580709` added covers that case:') with 'The plan rule `a69580709` added covers that case only when the card's intent spans the repository:' and add, after line 23, the sentence 'A card with a narrower intent that changes command help in a package it does not name is still first caught by a gate of scope `all`.'"
---

The range `3f83bb888aef38d1f9bd8d5d03c731661d6699d5..HEAD` is empty:
HEAD is the base commit, and `git log` and `git diff --stat` over it print nothing.
That matches both of the record's Added after Discussion entries, which settle the task as no code, no test and no `//lym:guard` mark.
The four lenses found nothing in the diff, and the change description is the one claim checked against the tree.

The description's "Why it no longer happens" chain holds.
`a69580709` is an ancestor of HEAD.
`internal/gatecli/scope.go:291` widens a `--card` scope through `impactset.DependentPackages`.
`cmd/lym/main.go:18` imports `internal/boardcli`, whose `storeVerbs` builds the `upsert` verb.
`TestCLIForm_EveryCommand` calls `tiergate.Require(t, 0)` at `cmd/lym/clitree_test.go:321`.
The hub's gate config sets `batch` to tiers 0 and 1 over scope `card` and leaves `card` at the template's tier 0 over scope `card`, so both gates run the test.

No commit carries a `Root-Cause:` trailer.
That is no finding, because the amended entry no longer asks this run to fix a bug.
The `two-way` merge-danger call holds for an empty diff.

### [LOW] Remaining gap overstates what the plan rule covers

**Location:** `_lym/landing/summary.md:21`

**Issue:** The section names the gap as "a card that changes command help in a package it does not name" and says the plan rule "covers that case".
The rule, as `contracts/specs/loom-plan-spec.md:366-367` and `contracts/stencils/loom/loom-rubric-plan-review.md:61-63` word it, binds only a card whose Intent covers every instance of one kind across the repository.
A card with a narrower intent that edits help in a package it does not name falls outside the rule, so the description claims more coverage than the rule gives.
The parent's decision asks the description to name the remaining gap the plan rule covers, so the fix narrows the claim to that and names the rest.

## Rejected

- No fork finding to reject: all four lenses returned no findings over the empty range.
