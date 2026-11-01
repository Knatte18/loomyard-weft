# Shorten the plan spec and split it into sub-files by need

`contracts/specs/loom-plan-spec.md` was 112 KB, so no agent could read it whole and every reader sliced it.
Half of it was the catalogue of `internal/planparser` validation checks, much of it restating code, history, and rationale and fixes the gate already prints.
The spec is now about 24 KB, and the rest is split into files that a role reads only when it needs them.

## What changes

- `loom-plan-spec.md` keeps the plan format: the card model, card fields and order, card types, handles, the shape classifier and the verify model.
  History and rationale a reader does not act on are cut.
- `loom-plan-checks-spec.md` (new, about 20 KB) holds the validation-check catalogue.
  `lym loom validate-plan` prints each finding with its check ID and way forward, so a reader opens an entry only when that finding does not settle it.
  It also says which layer raises each check: `planparser`, `planglyph`, `loomshed` or dispatch.
- `loom-plan-csharp-spec.md` (new) holds the rules that apply only under `language: csharp`.
- The part-scope and the join-of-parts rules move into `loom-parts-spec.md`, since only the parts flow reads them.
- `contracts/specs/specs.go` embeds the two new specs, and `specs_test.go` pins the new spec set and the stencils' copies of them.
- Every pointer follows the new files:
  the `loom-block-plan-*` stencils, the `loom-rubric-plan-review` rubric, the `webster-template-plan-fix` template, `docs/overview.md`, and comments in `internal/planparser`, `internal/planglyph` and `internal/webstercli`.
  Where a pointer named the old validation-checks section, it now names `loom-plan-checks-spec.md`, and the C# pointer names `loom-plan-csharp-spec.md`.
- Two stencil placeholders now say that `ImpactSummary` is required for `Edit`, `Extends` and `Delete`, matching the spec's per-type table.
- `webster-template-plan-fix` tells Webster's fixer where a named check is described and, for a C# plan, where the C# rules are.

## Design decisions

- The split is by reader and need, not by topic size: the writer's card format stays in one file, the check catalogue is read only on a refusal, and C# and parts rules are read only by plans that use them.
- The spec files remain durable contracts under `contracts/specs/`; no Go behavior changes.
  The Go edits are comments and one embed list.

## To check by hand

- Read the shortened `loom-plan-spec.md` for anything a writer still needs that went to a companion file or was cut.
- Compare `loom-plan-checks-spec.md` with the checks `internal/planparser/validate.go` and the `planglyph` package doc list, to confirm no check ID is missing or misattributed to the wrong layer.
- Search the repository for any remaining pointer to the old "Validation checks" section of `loom-plan-spec.md`.
- No Go test, gate or integration run was done for this description; run `lym gate test ./contracts/specs ./internal/planparser` before landing.
