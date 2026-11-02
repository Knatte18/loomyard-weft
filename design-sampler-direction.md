# The sampler: code writer or read-only discussion role, and the skills that follow

`sampler-direction` · Note · enhancement, loom, stencil

## Question

Does the sampler stay a code writer, or become a read-only discussion role as note `discussed-task-split` proposes?

## What follows from the answer

- Skills: `internal/samplerengine/spec.go` gives the sampler `scribe:prose` and `scribe:conversation` only.
  As a code writer it needs `scribe:code-quality`, `scribe:testing` and the language skills, as every other code-writing seat lists, the same gap as task `landing-fixer-base-skills` closes for the landing fixer.
  As a read-only role, `scribe:prose` and `scribe:conversation` may be enough.
- PATTERN role: the directive itself is note `sampler-lacks-pattern-directive`; this answer only decides whether it is rendered for the implementer or the Discussion writer.
- Bundle with the other small sampler changes once the direction is settled.

## Also read (orch, 2026-11-02)

- Shell guard: the standing Bash denies of `internal/shellguard` render into every Claude session's settings through `internal/shuttleengine/claudeengine/settings.go`, the in-place edit row among them, so the sampler's session has them.
- Heddle: the batching rules reach a session through the worktree's rendered `CLAUDE.local.md`, so the sampler in its task worktree should have them; not verified in a live sampler session.
