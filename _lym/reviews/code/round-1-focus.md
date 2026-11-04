---
round: 1
focus:
  - "internal/beamcli/land.go, internal/pawlengine/beamawait.go, internal/pawlengine/landrequest.go: does `lym beam reopen` refuse unless the run is at PR-Review or at a row after Beam-Await and before Finalize, and what state is left if a later step fails after an earlier one succeeded (PR drafted, flag removed, run moved)?"
  - "internal/landingshed/prreview.go, internal/pullrequest/client.go: does reopen stop the pr-reviewer, convert the PR to a draft, post the comment, and honour the close-instead flag; does the next PR-Review reuse the open PR and does Beam-Describe refresh its body?"
  - "Ready-flag removal in internal/pawlengine/ready.go and internal/beamcli: is `_lym/landing/beam-ready.json` removed and committed so the dobby cannot land the beam again, and does the run wait in Beam-Await rather than advance?"
  - "internal/landingshed/beamdescribe.go, finalize_pr.go: does the squash subject end with ` (#N)` as GitHub's squash does, and does Beam-Describe list every sub-PR merged into the beam branch oldest first with number link, title, merge time and summary, with direct commits (including parent merges) interleaved in order with a commit link?"
  - "How Beam-Describe reads PR numbers from the beam's commit subjects: subjects without a suffix, with a suffix inside the title, or a parent-merge commit; are they classified as PR or direct commit correctly?"
  - "Recipe and wiring changes (contracts/recipes/pawl-recipe.yaml, internal/loomcli/beamwiring.go, landingdeps.go, internal/hubgeom/dobbygeom.go): do the new seams get filled in production wiring, and do the docs the repository names for these changes (doc.go files, docs/overview.md) land in the same diff?"
  - "Test economy: new tests in land_test.go, prreview_test.go, beamawait_test.go, client_test.go and beamwiring_integration_test.go against existing coverage, and behavior added with no test."
---

Seed focus from the decision record's two parts (reopen verb, PR number in squash subjects plus the Beam-Describe list) and the diff's file spread.
