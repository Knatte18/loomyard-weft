# Render the PATTERN directive into the sampler agent's prompt

The sampler agent writes and commits code with the operator in an interactive session, yet its prompt carried no PATTERN directive.
Every other code-writing role gets one, so a sampler session never saw the PATTERN rules, the abscissa rule among them.

`samplerengine.SessionSpec` now renders `pattern.Directive` with `pattern.RoleImplementer` from the task worktree's PATTERN overview.
The `sampler-template-agent` stencil gains a `pattern_directive` marker, placed after the opening role paragraph and before `## Start`.

## Design decisions

- The marker is optional.
  The prompt is filled with `stencil.FillOptional`, so a worktree with no PATTERN overview renders an empty directive instead of failing the session.
  The four other markers stay required.
- The role is `pattern.RoleImplementer`, since the sampler writes code today.
  Should the sampler become a read-only discussion role (note `sampler-direction`), only the role passed changes, to the Discussion writer's.
- The prompt was paths-only before.
  It now also carries the PATTERN overview inside the directive.
  The stencil header comment, the `SessionSpec` doc and the `samplerengine` package doc say so.

## Tests

`TestSessionSpec` gains a case that writes a PATTERN overview into a temporary worktree and checks that the spec's prompt contains the implementer directive for it.

## To check by hand

- Open a sampler session in a worktree that has a PATTERN overview and confirm the agent sees the directive.
  This was not run interactively.
