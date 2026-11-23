```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: no findings.

I tried to build a case where dropping the "reviewer model differs from the previous round's" clause is wrong, and none held.

- **Rotation case:** `Rotate` (`internal/hardenshed/rotate.go:7-15`) cycles the list, so a list of two or more entries gives each round a model that differs from its predecessor. `CheckCampaign` already refuses lists that put the same model in neighbouring rounds, wrap-around included, and the review states this. Under a rotation the dropped clause was always true, so removing it loses no check.
- **One-model default:** `template.yaml:5` sets a one-entry list, so every round's reviewer is `sonnet[high]`. With the clause kept, a default campaign could never converge. Removing it is the only change that lets the default campaign finish.
- **Leftover wording:** I grepped `contracts/stencils/harden` and `internal/hardenshed`, excluding tests, for `differs`, "all three", "different model", "same model" and "model differs". None remain.
- **Judge facts:** the `Models:` line in `campaign_rows.go:186` still prints the models, and the judge no longer needs it to rule on convergence. The "reads both" wording matches the two conditions left in the bullet.

Tool calls made (1 of 6):
1. `git grep` for rotation and model-difference wording in the harden stencils and `internal/hardenshed`. My context held only the one edited stencil span, so I needed it to check that no other text still assumes the clause and to confirm the rotation mechanics.
