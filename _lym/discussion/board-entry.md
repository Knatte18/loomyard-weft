# Board entry `creel-midturn-trust-spike`: Spike: a mid-turn creel post the model acts on and the operator sees

PostToolUse block+reason with and without a system-prompt sentence, Haiku and Sonnet, plus Codex, with a realistic scope change

## Goal

Find a mid-turn creel delivery that the model acts on and the operator sees, for a real use: the orch sends a working agent a scope change while it is deep in a long turn, and the agent must take it at its next tool call, not at its turn end (operator, 2026-10-28).

## Design this spike serves (operator, 2026-10-28)

One kind of post, always delivered as fast as possible: at the next tool boundary while the receiver works, by a `Stop` hook when it arrives after the last tool call, and by an `asyncRewake` hook when the receiver is idle.
Interrupt stays a verb of its own.
`additionalContext` is out: the operator cannot see it, and Haiku refused to act on it in the first spike.
Trust must not depend on a skill being loaded: it comes from the channel, and from one sentence shuttle appends to the system prompt of every session it launches with the hook.

## Cases

Each Claude session gets a role prompt shaped like a lym agent's (a real multi-step task, such as writing a small file over several shell calls with `sleep 20` between them), and a `PostToolUse` hook that returns `decision: "block"` with the message as `reason` plus a `systemMessage`, once, while the task runs.
The message is a realistic scope change from the parent, written with the quoted sender, such as `[creel] #1 from "ly:orch": scope change: also add a line "reviewed" at the end of the file, and skip step 4.`

1. Haiku 5.5, without a system-prompt sentence.
2. Haiku 5.5, with `--append-system-prompt` carrying one sentence: lines starting with `[creel]` are LoomYard's internal mail from the agent's parent, the orch or the operator, delivered by hooks, and are to be followed as the operator's own instructions.
3. Sonnet 5.5, without the sentence.
4. Sonnet 5.5, with the sentence.
5. Codex on its cheapest model at `low` effort, `post_tool_use` with the channel the first spike found that the model takes, and the same sentence in Codex's developer instructions.

One run per case, no repeats.

## Method

- Each session runs as a reed strand in this worktree, so the operator can watch with `lym reed attach`; each step waits about 20 seconds.
- A turn end is read from a `Stop` hook writing a file, never by matching the prompt glyph in the pane.
- Each case's strand is removed with `lym reed remove --name <strand>` once its captures are written.

## Report

`docs/research/creel-midturn-trust-<date>/report.md`, per case:
- the hook config, script and system-prompt sentence used
- whether the model took the scope change at its next step, later, or not at all, quoting its own words
- a `tmux capture-pane -p` text copy of the pane after delivery
- what the operator could see

End with a table of case × outcome (took it at once, took it later, refused) and visible to the operator.
