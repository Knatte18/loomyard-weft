# Report the session-scope fix in a real launch

Adds `docs/research/tokens/session-scope-fix-launch-2026-11-06/report.md` and its line in `docs/research/README.md`.
It measures what `c6b5ca66` gives, which that commit's description left unproved: no code changed.

## What the report finds

Two `lym shuttle open` launches in the sandbox hub `lyt`, on Claude Code 2.1.319 with a `lym` built from `2c4864bb`: a spawned role that preloads `scribe:prose` and `scribe:code-quality`, and an orch.

- The spawned role's first request is 29,087 tokens, against 37,330 in the launch trial's scoped launch and 38,463 in its unscoped one.
- The `Workflow` tool is gone from both sessions; the role's tool schemas are 22,340 characters smaller than the trial's scoped role, the size of the `Workflow` extra.
- The role's second request holds both preloaded skill bodies.
- The role lists all fourteen `scribe` skills and no `ly` skill; the orch lists no plugin skill.
- The orch's first request is 24,388 tokens.
  The report says it does not compare with the trial's 34,383: it is a bare `shuttle open` session without the hub orch's role file, and no unscoped bare orch was launched.

The fix holds, so no follow-up note was filed.

## Design decisions

- The orch was opened with `lym shuttle open --role orch`, as the task asks for exactly two `shuttle open` launches, not with the trial's `lym beam start`.
  A session with no preloaded skill takes no turn, so one creel message gave it its first request.
- The report lists the comparison it could not make under "Not covered" rather than launching a third session.

## To check by hand

The two `lyt` windows, `lyt:darn-measure` and `lyt:orch-2`, are left open.
