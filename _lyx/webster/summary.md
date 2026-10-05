# test-writing-rules: plan spec, stencils and rubrics follow PATTERN-test-economy

All four batches recorded `done`, each committed by its own fork.
Every batch's own gate (`go build`, `go test`, the integration tests of the touched package) passed.
The only warning, on batches 2 to 4, was informational: `contracts/stencils#wordingClaims` in `claims_test.go` was touched outside the batch's own target glyphs.
That is the intended mechanism: the wording pins move as rows and claims of the existing table.

- **01-plan-spec-card-model** (`0cfbfb430`): `contracts/specs/loom-plan-spec.md`.
  The card criterion says a card carries the coverage for new behavior.
  Card granularity follows the public surface instead of testability.
  The worked-example intro describes card 2 as the multi-label example only.
  The fenced card bodies are unchanged.
- **02-plan-template-coverage** (`dcd4b9aad`): `loom-template-plan.md` asks for coverage instead of a bundled test file.
  Extending an existing test file is the normal shape, and a `**Create:**` test file is for an uncovered surface.
  `claims_test.go` pins the new phrases and bars the old ones.
- **03-review-rubrics-test-economy** (`8a41b4205`): the Plan-Review rubric grades granularity by public surface and gains a test-economy bullet.
  The Webster-review rubric gains a test-economy bullet whose findings must name the existing test that grounds them.
  The claims were updated to match.
- **04-fix-and-discussion-stencils** (`524746a28`): `burler-step-3-fix.md` adds a test only for a coverage gap.
  `loom-template-discussion.md` says a must-cover scenario names a behavior, not a test.
  Each has one new claim.

No deviations from the declared file-ops were reported, and no new `Test` function was added.
The plan-level verify gate runs after this turn ends.
