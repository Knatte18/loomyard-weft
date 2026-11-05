# Report the live trial of creel mail reaching Codex sessions

This adds the report of a live trial in the sandbox hub `lyt`: real Codex sessions launched through `lym shuttle run` with the build that carries `codex-creel-hooks` take creel mail as Claude sessions do.
No code changes; the report is `docs/research/codex/engine-trial/codex-creel-trial-2026-11-05.md`, linked from the research index.

## What the trial found

- Mail posted while a session runs a tool arrives at the end of that tool call through `PostToolUse`, 0.07 s after the call ends, and the model acts on it.
- Mail posted as a turn ends arrives through `Stop`, the session continues the turn and answers it.
- Mail to an idle session wakes it through `codex queue`; the turn starts 0.2 to 8.8 s after the post.
- A session answers an agent's mail that expects a reply with `lym creel post --reply` from inside its `workspace-write` sandbox, and posts mail itself from there.
- Three mails in a row, and two sessions at once, each reach only their own session, in order, both idle and mid-tool.

## Design decisions

- The first session of the trial ran on the previous `lym` binary, since the landed build was installed one second after its launch; the report records it as not counted and every case ran on sessions launched afterwards, checked for `hook-read` on the Codex command line.
- A notice from a lym program was not reproduced, since no trial session may impersonate a lym sender; the report says so and rests on the EXTERNAL notice.
- The one fault, a reminder line that starts a wasted turn on a strand with a wake queued, is filed as board note B2 in project `codex-engine-trial`.
