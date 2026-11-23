# Agent sits idle after its skill-load turn; its prompt is never typed

`prompt-not-typed-after-skills` · Note · bug, shuttle

## Seen

2026-11-23: `ly:godocreflow-worsens-findings:code-burler-fix` sat idle for 30 minutes at Code-Burler-Fix.
Dobby sent its idle notice; asked by the orch, the agent answered that it had been idle after loading its skills and only then started reading its prompt file.
So the task prompt was never typed after the skill-load turn, and only the orch's creel message woke it.
The same morning the API answered 529 Overloaded to other agents, and LOOM reported another agent held at its skill-load turn by API errors; the two may share a cause.

## Expected

Once the skill-load turn ends, the agent's prompt is typed, or a failure to type it is retried and noticed, so no agent waits for a parent's message to start.


## From issue #195

Imported from [issue #195](https://github.com/Knatte18/loomyard/issues/195).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `/.lym/shuttle/`.

## Seen

A tapestry run's Plan-Bouncer seed session got only the skill bootstrap ("Load these skills now … scribe:prose …"), answered it, and then sat at an empty prompt.
The line pointing it at its `prompt.md` never arrived.
The session's `events.jsonl` shows `SessionStart`, one `UserPromptSubmit` and `Stop` within three seconds, and nothing more.

It happened twice in a row on the same row:

- Session `8a36796b…`, spawned at 11:00:20, idle from about 11:00 to 11:04.
- Session `724ef172…`, spawned at 11:04:32 (lym's own respawn), with the same pattern: `SessionStart` 10:04:32Z, `UserPromptSubmit` 10:04:33Z, `Stop` 10:04:35Z.

`lym loom status` showed `current_producer: Plan-Bouncer` with empty `error` and `wait` throughout, and no notice reached the parent orch.
The operator noticed the stall by watching the pane.
`lym loom goto --to Plan-Bouncer` was refused as `shed_busy` because the live driver held the run lock.

## Workaround

The parent orch posted the missing line to the strand by creel:
`Read <worktree>/.lym/shuttle/<id>/prompt.md in full first; it is your complete, authoritative instructions.`
The session took it as its next `UserPromptSubmit` and started working.

## Expected

The task prompt reaches the session after the skill bootstrap every time.
If a session ends its bootstrap turn and no task prompt follows within a short bound, lym delivers it again or raises a notice to the parent.
A respawn on that row does not repeat the same failure.
