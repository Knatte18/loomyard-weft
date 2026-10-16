---
format: 5
approved: false
---

# Plan: loom-stencil-wording

Fix the loom and shed stencil wording the stencil intent audit found wrong (D1–D4), and move the plan writer, Plan-Review and webster implementer prompts off this repository's Go verify tiers onto a hub-wide `verify_tiers` list in `language.yaml` (D5–D7).
Card 1 adds the key and its one rendering; Cards 2–4 render it at the plan, rubric and implementer sites; Cards 5–7 are the prose fixes and the spec.

## Card Index

1 — language-verify-tiers — `language.yaml` gains `verify_tiers`; `Load` returns both keys; `VerifyTiersValue` renders the marker value
2 — plan-verify-tiers — the plan prompt renders the list, drops the Go tiers and never asks the operator
3 — rubric-verify-tiers — `ReadRubric` fills the list from the Bouncer and Burler rows; the `Verify coverage` bullet checks against it
4 — implementer-verify-tiers — both implementer renders carry the list; the raw-run section names raw test runs
5 — rework-findings-are-claims — rejection findings are a reviewer's claims, routed through the parent when they overturn a Decision
6 — driver-advisor-wording — the driver names no hand commit; the advisor reads the draft record
7 — spec-verify-model — the spec's verify model names `verify_tiers` and `publish_verify`

## Shared Decisions

### One marker value

Every site fills the same `verify_tiers` marker with `languageconfig.VerifyTiersValue`: the entries as `- ` bullets, verbatim and in order, or `none` for the empty list.
Every sentence around it, each fallback included, lives in the stencil (`PATTERN-stencil-ownership`), and the marker is required wherever it appears.

### Readers of the unfilled stencils

The plan writer's lens forks read the Plan-Review rubric by path, and the rework writer reads the plan stencil by path, both unfilled; D6 names no such site, so neither gains a fill.
The plan writer's own filled prompt, which the lens forks inherit, carries the list.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
