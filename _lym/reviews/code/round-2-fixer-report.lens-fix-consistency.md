```yaml
lens: fix-consistency
outcome: reported
```

Task: the fix-consistency lens over this round's change set (commits 0a4519673..109867145, plus the `_lym/landing/summary.md` edit).

1. **LOW**: `internal/beamcli/dobbydeps.go:349-350`, the twin of the F3 budget change, left as it was.
   - The comment reads "The describer agent runs on landing.yaml's describe model for its describe_timeout_min. The Describe row's session runs on the same two keys."
   - While a reviewer runs, which is the template default, the Describe row's runs are now limited to `describe_timeout_min` plus `describe_review_timeout_min`. The second sentence no longer holds for the timeout, and the beam describer still gets `describe_timeout_min` alone.

2. **LOW**: `_lym/landing/summary.md:21`, not updated for F5.
   - It says "The Describe writer's prompt states the same rule for the description."
   - The beam describer's prompt (`contracts/stencils/beam/beam-template-describe.md`) now states that rule too. The change description names the beam describer's gate and recorders (line 20) but not its prompt.

The other cross-references I checked agree with the change:
- the `template.yaml` comments, the `config.go` field docs, `docs/overview.md`, and the `describereview.go` and `describe.go` docs on the new budget;
- the new Warn against the `judge` doc (`prreview.go:442`) and summary line 45;
- the tooling-name list in the beam stencil against the describe and describe-review stencils.

Tool calls (3 of 6):
- A grep of docs, contracts, internal and the summary for budget, timeout and unbuildable-text wording: my context lacked the sibling mentions.
- The describe-review stencil and a grep for the tooling-names rule: my context did not hold that stencil or the other places that state the rule.
- A grep of `internal/landingshed/doc.go` and the final-summary spec's forbidden-terms section: my context held neither.
