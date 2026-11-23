---
round: 2
focus:
  - "contracts/stencils/harden/harden-rubric-review.md, under `## The judge`: does any rule, in the Converged bullet or elsewhere in the stencil, still require the reviewer model to change between rounds, so that a campaign whose `reviewer` list in `internal/hardenshed/template.yaml` holds one entry could not converge?"
  - "Other docs and stencils that describe hardener rounds (grep `contracts/` and `internal/hardenshed` for reviewer rotation or model change): does any text still present rotation as the default rather than a per-run setting?"
---

The one-model default is `reviewer` and `fixer` set to `["sonnet[high]"]` in `internal/hardenshed/template.yaml`.
`CheckCampaign` refuses lists of two or more entries that put one model in neighbouring rounds.
