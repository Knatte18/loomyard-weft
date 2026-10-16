# Plan and Plan-Review stencils hard-code this repository Go verify tiers

`stencil-verify-tiers-from-config` · Note · bug, stencil, loom

## Seen

Found by the review of the stencil intent audit of 2026-10-15; line numbers as of `main` on 2026-10-16.

Hub-generic stencils name this repository's Go verify tiers:

- `contracts/stencils/loom/loom-template-plan.md:201-205` prescribes the `## verify:` order (`go build ./...`, `go vet` under `-tags integration`, `-tags tmux` and `-tags llm`, the untagged `go test` tier, then the `integration` tier), the `llm` and `tmux` tag rules, `go -C <module>` for a nested module, and that Publish runs the `tmux` tier through `publish_verify`.
- `contracts/stencils/loom/loom-rubric-plan-review.md:93-96` repeats the `integration`, `llm` and `tmux` tag rules and the order check.
- `contracts/stencils/webster/webster-body-implementer.md:80` allows "raw `go test`" on the batch's packages.

lyx must not hard-code Go: a .NET or Python hub gets this repository's Go tiers in its plan writer and Plan-Review prompts.
`dotnet-verify-protocol` and `dotnet-scoped-verify` cover how a .NET hub verifies, not the Go text in these stencils.

## Decision

Operator decision (2026-10-16): the finding stands and gets its own note.

## Idea

Agent's suggestion: the tier order, tag names and their compile-or-run rule come from the hub's config (`language.yaml`, or landing config for the tier Publish runs), and the stencils render them instead of naming Go; check the result against `dotnet-verify-protocol` so the .NET tiers fit the same config.
