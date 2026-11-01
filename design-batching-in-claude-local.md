# Trial: the batching instruction in the rendered CLAUDE.local.md through heddle

`batching-in-claude-local` · Task · enhancement, claude, heddle · running · Describe

## Seen (orch, 2026-11-01)

The batching sentence every shuttle-launched Claude session gets ("Request every call whose arguments you already know in one message, …", `batchingSentence` in `internal/shuttleengine/claudeengine/settings.go`) sits at the end of Claude Code's long system prompt, which already asks for parallel tool calls itself.
After `turn-batching` landed, Webster forks went from 1.37 to about 1.7 read, grep and edit calls a turn (`tools/forkaudit`, 27 forks of `warp-visibility`, `interactive-recipe` and `run-verbs-shared`).
Claude Code puts a project's CLAUDE files into the conversation as a context block under "These instructions OVERRIDE any default behavior and you MUST follow them exactly as written", a stronger frame than an appended system-prompt sentence; another user reported cutting token use by putting such an instruction there.

## Idea (operator, 2026-11-01)

Test it with heddle, with no code change: a heddle entry aimed at the `claude-file` target puts the batching instruction into the rendered `CLAUDE.local.md` at every worktree's root, which every Claude session in the hub reads, Webster's forks through Merriam's context included.
After two or three tapestry runs, `tools/forkaudit` measures calls per turn against the 1.7 above.
The test is best run after note `edit-with-edit-tool` has landed, or measured on read and search calls alone, so that the edit change does not confound it.
