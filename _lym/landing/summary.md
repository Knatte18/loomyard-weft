# The sampler agent's prompt carries the PATTERN directive

The sampler agent writes and commits code, yet it was the one code-writing role whose prompt carried no PATTERN directive, so an interactive sampler session never saw the repository's PATTERN rules.

## What changes

- `samplerengine.SessionSpec` renders `pattern.Directive` with `pattern.RoleImplementer` from the task worktree, as the darn writer and the Drift-Verify implementer seat do, and fills it into the stencil through `stencil.FillOptional`.
- `sampler-template-agent.md` gains an optional `pattern_directive` marker, placed after the role's opening paragraph and before `## Start`, so the agent reads the rules before it touches code.
  A worktree with no `PATTERN.md` renders an empty directive and the prompt builds as before.
- The package doc and the stencil's banner name the new marker.

## Design decisions

- The role is `RoleImplementer`, since the sampler writes code today.
  Should the sampler become a read-only discussion role (note `sampler-direction`), only the role passed changes.
- No declared PATTERN changes are passed: the sampler has no plan or record section that declares them.

## Tests

`TestSessionSpec` gains a case where the task worktree holds a `PATTERN.md`: the prompt carries the implementer directive rendered from it.
The existing valid case, whose worktree has no overview, pins that an absent `PATTERN.md` still builds the spec.
