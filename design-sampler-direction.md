# The sampler: code writer or read-only discussion role, and the skills that follow

`sampler-direction` · Note · enhancement, loom, stencil

## Question

Does the sampler stay a code writer, or become a read-only discussion role as note `discussed-task-split` proposes?

## What follows from the answer

- Skills: task `landing-fixer-base-skills` landed (`7cffa111`) and, as a same-family fix, gave the sampler `scribe:code-quality` and `scribe:testing` beside `scribe:prose` and `scribe:conversation` in `internal/samplerengine/spec.go`, on the footing that it writes code and tests.
  The language skills are still not listed for it, as other code-writing seats list them.
  If the sampler becomes a read-only role, the two added skills may be more than it needs and can come out again.
  The landing's review was asked to confirm the sampler change belongs there.
- PATTERN role: built by task `sampler-lacks-pattern-directive` (`9557b4fa`).
  `samplerengine.SessionSpec` renders `pattern.Directive` with `pattern.RoleImplementer`, since the sampler writes code today.
  If the sampler becomes a read-only role, only the role passed changes, to the Discussion writer's.
- Bundle with the other small sampler changes once the direction is settled.

## Also read (orch, 2026-11-02)

- Shell guard: the standing Bash denies of `internal/shellguard` render into every Claude session's settings through `internal/shuttleengine/claudeengine/settings.go`, the in-place edit row among them, so the sampler's session has them.
- Heddle: the batching rules reach a session through the worktree's rendered `CLAUDE.local.md`, so the sampler in its task worktree should have them; not verified in a live sampler session.
