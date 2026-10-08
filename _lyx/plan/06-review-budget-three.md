# Card 6 — review-budget-three

**Edit:**
- `internal/loomengine/template.yaml`
- `internal/loomengine/config_test.go`
- `internal/testkit/envkit/envkit.go`
- `internal/loomrecipe/escalation_test.go`

**Intent:**
Drop the template review budget from five rounds to three (D3); a spent budget still escalates to the parent exactly as today.

- `template.yaml`: `review_max_bounces: 3`, and both its comment and the `review_circling_checkpoint` comment state that with both at 3 a run reaches the budget escalation at round 3 before a CIRCLING verdict could change the outcome, and that lowering the checkpoint gives an earlier circling escalation.
  `review_circling_checkpoint` stays 3.
- `config_test.go`: the template-load expectation pins `ReviewMaxBounces` at 3.
- `envkit.go`: the `ReviewMaxBounces` default that mirrors the template follows it to 3.
- `escalation_test.go`: the budget the escalation tests run with becomes 3, so a CONTINUE at round 3 escalates with cause `budget`, pinning the template's budget end to end.

Grep the tree for any other test pinning the template's old value of 5 and move it with this card.

**ImpactSummary:** Template default and its mirrors only; runs whose loom.yaml already sets review_max_bounces keep their value.
