## Goal

```markdown
## Seen (operator, 2026-11-02)

An interactive sampler agent told the operator that PATTERN reaches only the loom agents, not an interactive session: working directly with Claude in the code, it does not see the abscissa rule.

## Cause (orch, from the source)

`internal/samplerengine` and `contracts/stencils/sampler/sampler-template-agent.md` name no PATTERN directive, and the stencil's markers are `slug`, `task_worktree`, `decision_record_path`, `parent_directive` and `edit_directive` only.
Every other code-writing role renders `pattern.Directive` into its prompt: Webster (`internal/websterengine`), darn (`internal/darnengine/spec.go`), Drift-Verify (`internal/driftengine/seats.go`), the Discussion and Plan writers (`internal/loomengine`) and the review seats (`internal/burlerengine`).
The sampler is the one role that writes and commits code and gets none.

## Fix

Render the PATTERN directive into the sampler's prompt with `pattern.Directive`, as `VerifyTable` does for its implementer seat, and add a marker for it to the stencil; the role is `pattern.RoleImplementer`, since the sampler writes code today.
Add a test that the sampler's prompt carries the directive.
The directive is needed whatever the sampler's direction (note `sampler-direction`): should it become a read-only discussion role, only the role passed changes, to the Discussion writer's.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
