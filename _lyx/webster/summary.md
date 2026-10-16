# loom-stencil-wording: hub-wide verify tiers and stencil wording fixes

All four batches recorded `done`; the plan-level verify gate runs after this turn.

- **01-language-verify-tiers** (head `b51a45dd`): `language.yaml` gained `verify_tiers`, and `Load` returns both keys.
  `VerifyTiersValue` renders the marker value.
  Scope guard deviations: `internal/languageconfig/config.go` and `internal/loomcli/cli.go`.
  The fork reported adding a `Language` field on `loomCLI` so `wire`'s kept value can be asserted by a test.
- **02-plan-verify-tiers** (cards 2 and 3, head `65524dd1`): the plan prompt renders the list, drops the Go tiers and never asks the operator.
  `ReadRubric` fills the list from the Bouncer and Burler rows, and the `Verify coverage` bullet checks against it.
  No deviations.
- **04-implementer-verify-tiers** (head `5b54b8ff`): both implementer renders carry the list, and the raw-run section names raw test runs.
  Scope guard deviation: `internal/websterengine/template_test.go`.
  The fork narrowed two existing pack-row needles that matched a bare `none` globally, and raised the `contracts/stencils` row in `test-budget.yaml` to 17.
  The new scan test is not in the card's guard `-run` list, so only the unit tier ran it.
- **05-rework-findings-are-claims** (cards 5, 6 and 7, head `dc75a3c8`): rejection findings are a reviewer's claims, routed through the parent when they overturn a Decision.
  The driver names no hand commit, the advisor reads the draft record, and the spec's verify model names `verify_tiers` and `publish_verify`.
  No deviations.

I did not run the plan-level `## verify:` command; the gate does.
