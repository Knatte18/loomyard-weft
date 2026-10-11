# Plan-Review round 1 — fixer report

## Changes

- F1 (fixed as proposed): Card 3 now lists `contracts/stencils#wordingClaims` under `**Edit:**`.
  Its Intent rewords two claim whys, the `loom-rubric-plan-review.md` row's `is not part of the answer key` and the `loom-template-rework.md` row's `never plan an entry as scope`, so each describes an entry's findings as fixed or disputed and unseen by a fresh reviewer, with the `must` phrases unchanged.
  The ImpactSummary and Card 3's Card Index line name the two whys.
  The finding was confirmed against `contracts/stencils/claims_test.go:138` and `:171`, and a repo-wide grep found no other site with the old wording.
- F2 (fixed as proposed): Card 1's sentence telling each why to cite D1 to D3 of the decision record is replaced by one telling each why to state the rule its phrase pins, as the neighbouring rows do.
  Confirmed: `git ls-files _lyx` is empty, and no existing why cites a decision-record entry.
- F3 (fixed as proposed): the `one-rule-many-sites` Shared Decision now says the orchestrator's Validate step names its two checks and points at instruction 3 without stating the rule.
  The other sites keep stating or pinning the stencil text.

`lyx loom validate-plan` reports `ok: true` after the edits.
The `approved:` key was not touched.

## Disputed

None.

## Deferred

None.
