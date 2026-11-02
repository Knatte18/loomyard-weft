# The sampler agent gets no PATTERN directive

`sampler-lacks-pattern-directive` · Note · bug, loom, stencil, pattern

## Seen (operator, 2026-11-02)

An interactive sampler agent told the operator that PATTERN reaches only the loom agents, not an interactive session: working directly with Claude in the code, it does not see the abscissa rule.

## Cause (orch, from the source)

`internal/samplerengine` and `contracts/stencils/sampler/sampler-template-agent.md` name no PATTERN directive, and the stencil's markers are `slug`, `task_worktree`, `decision_record_path`, `parent_directive` and `edit_directive` only.
Every other code-writing role renders `pattern.Directive` into its prompt: Webster (`internal/websterengine`), darn (`internal/darnengine/spec.go`), Drift-Verify (`internal/driftengine/seats.go`), the Discussion and Plan writers (`internal/loomengine`) and the review seats (`internal/burlerengine`).
The sampler is the one role that writes and commits code and gets none.

## Fix

Render the PATTERN directive into the sampler's prompt with `pattern.Directive`, as `VerifyTable` does for its implementer seat, and add a marker for it to the stencil; the role is `pattern.RoleImplementer` while the sampler writes code.
Add a test that the sampler's prompt carries the directive.
If the sampler becomes a read-only discussion role (see note `discussed-task-split`), it needs the directive too, since what it proposes must not break PATTERN; the role then follows the Discussion writer's.
Bundle with the other small sampler changes when the operator has settled the sampler's direction; do not offer this note alone as a task.

## Also read in the same pass (orch, 2026-11-02)

- Skills: `internal/samplerengine/spec.go:24` gives the sampler `scribe:prose` and `scribe:conversation` only.
  It writes and commits code but loads no `scribe:code-quality`, no `scribe:testing` and no language skills, the same gap as note `landing-fixer-base-skills`'s task, and every other code-writing seat lists the base skills.
  If the sampler stays a code writer it needs them; if it becomes read-only, `scribe:prose` and `scribe:conversation` may be enough.
- Shell guard: the standing Bash denies of `internal/shellguard` render into every Claude session's settings through `internal/shuttleengine/claudeengine/settings.go`, the in-place edit row (`sed -i`, `perl -i`, a heredoc or redirect into a source file) among them, so the sampler's session has them.
  The guard is a guardrail and not a barrier, and `sed -n` is a read it never matches; using it is a style miss against the rule to read with `Read` slices and `grep`, not a breach.
- Heddle: the batching rules reach a session through the worktree's rendered `CLAUDE.local.md`, so the sampler in its task worktree should have them; not verified in a live sampler session.
