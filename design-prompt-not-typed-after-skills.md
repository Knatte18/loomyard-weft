# Agent sits idle after its skill-load turn; its prompt is never typed

`prompt-not-typed-after-skills` · Note · bug, shuttle

## Seen

2026-11-23: `ly:godocreflow-worsens-findings:code-burler-fix` sat idle for 30 minutes at Code-Burler-Fix.
Dobby sent its idle notice; asked by the orch, the agent answered that it had been idle after loading its skills and only then started reading its prompt file.
So the task prompt was never typed after the skill-load turn, and only the orch's creel message woke it.
The same morning the API answered 529 Overloaded to other agents, and LOOM reported another agent held at its skill-load turn by API errors; the two may share a cause.

## Expected

Once the skill-load turn ends, the agent's prompt is typed, or a failure to type it is retried and noticed, so no agent waits for a parent's message to start.
