# Card 13 — plan-verify-guidance

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Edit:**
- `contracts/stencils/claims_test.go`

**Intent:**
Decision D4's plan guidance: a plan's `## verify:` no longer runs the `tmux` tier, which Publish now runs through `publish_verify`.

- `loom-plan-spec.md`'s verify model: the section runs, in this order, `go build ./...`, `go vet` under each of `-tags integration`, `-tags tmux` and `-tags llm`, the untagged tier (`go test`), then the `integration` tier (`go test -tags integration`).
  The `tmux` tier is run at Publish by landing config's `publish_verify` where the repo sets it, never by the plan.
  The sentence leaving the `tmux` gate to the board note `test-tiers` goes.
  The verify model also says that the Webster-Burler round gate runs a derived impacted-set command between rounds while Webster's gate, Publish and Finalize run the section itself, and that Publish runs `publish_verify` after it.
- `loom-template-plan.md`: the sentence that a plan's `## verify:` may run the `tmux` tag becomes the ordering above, with the `tmux` tier compiled by `go vet -tags tmux` rather than run.
- `loom-rubric-plan-review.md`'s verify-coverage bullet: a section that compiles the `tmux` tier rather than running it is not a finding; a section out of the stated order is.

`claims_test.go`'s claims on the two stencils follow the new wording: the `tmux` claims now pin that the tier is compiled rather than run by the plan, and the template gains a claim pinning the build, vet, untagged, `integration` order.

**ImpactSummary:** Wording of one spec and two stencils and the claims that pin them; future plans' `## verify:` sections change, no code does.
