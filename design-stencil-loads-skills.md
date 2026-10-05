# Stencils make sessions load scribe:conversation and scribe:prose

`stencil-loads-skills` · Note · enhancement, stencil, orch

The scribe plugin's `SessionStart` hook asks a session to load `scribe:conversation` and `scribe:prose`, but a hook can only ask: on 2026-10-05 the orch session saw the request and did not load either skill.

Idea (operator): lyx makes the loading imperative itself.
The stencils lyx renders for its Claude sessions open with an injected instruction to load both skills before anything else, so no session depends on the plugin hook being obeyed.

- The orch prompt carries it first.
- Settle which spawned roles need it too: every role that writes prose for the operator or into the repo.
