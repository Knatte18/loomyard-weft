```yaml
lens: fix-affected-set
outcome: reported
```

Task: affected-set check of the F1–F5 fixes.

Unchecked: none.

no findings

I reran one search per finding:
- **F1:** the key-path literal now appears only in the constant, the yaml struct tag and tests.
- **F2:** `syncText` holds the only unbuildable-text Warn; the entry-call failure returns an error.
- **F3:** the places that state the Describe run budget agree on the sum. These are the template comments, the `config.go` docs, the `describeReviewSeat` and `DescribeTable` docs, and `docs/overview.md`. The reviewer stencil states no figure. `beamcli/dobbydeps.go` runs no reviewer, so its use of `describe_timeout_min` alone still holds.
- **F4:** the beam forwarding sites are `dobbydeps.go` (gate and findings) and `deps.go` (both recorders and `DescriptionFindings`), and each now has a test.
- **F5:** the stencils that write a change description are darn-write, landing-describe and beam-describe, and all three now state the rule.

Tool calls (5 searches by finding, 1 follow-up):
1. Grep for the quoted key literals: my context did not show every Go site.
2. Grep for statements of the Describe budget: sites outside the files already read.
3. Grep for `ForbiddenTerms` and `ValidateDescription` callers and tests: the full set of forwarding sites was not in context.
4. Grep of stencils for pull request and description wording and the tooling-names rule: those stencils were not read.
5. Grep for unbuildable-text and title/body sites: sites outside `prreview.go` were not in context.
6. Grep for description-writing stencils and timeout wording in the reviewer stencil and docs: neither had been read.
