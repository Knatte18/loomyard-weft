## Goal

Find a mid-turn creel delivery that the model acts on and the operator sees, for one real use: the orch sends a working agent a scope change while it is deep in a long turn, and the agent takes it at its next tool call, not at its turn end.
The deliverable is a research report, `docs/research/creel-midturn-trust-2026-10-28/report.md`; no lym code changes.

## Scope

- Five cases, one run each: Haiku 5.5 and Sonnet 5.5 each with and without one system-prompt sentence appended through `--append-system-prompt`, and Codex on its cheapest model at `low` effort with the same sentence in its developer instructions.
- Each Claude case uses a `PostToolUse` hook returning `decision: "block"` with the message as `reason` plus a `systemMessage`, once, while a multi-step task runs.
- Codex uses `post_tool_use` with the channel the first spike found the model takes (`additionalContext` plus `systemMessage`).
- Each session runs as a reed strand in this worktree; a `Stop` hook writing a file marks every turn end.
- The report gives per case the hook config, script and sentence, whether and when the model took the scope change with its own words quoted, a pane capture after delivery, and what the operator saw, then a case-by-outcome table.
- Out of scope: any lym code, creel or shuttle change, and the idle (`asyncRewake`) and `Stop` channels, which the first spike covered.

## Decisions

- **Task shape.** Each case's prompt asks for a file written over several shell calls with `sleep 20` between them, in a role-prompt shape like a lym agent's.
  Rationale: the spec asks for a realistic multi-step task; the sleeps give the hook a window to fire mid-turn.
- **Delivery timing.** The hook fires once, after the second tool call, tracked in a count file.
  Rationale: one message at a known boundary lets "at its next step" be read from the transcript.
- **Message.** `[creel] #1 from "ly:orch": scope change: also add a line "reviewed" at the end of the file, and skip step 4.`
  Rationale: a scope change that alters both the file and the remaining steps is checkable from the file and the tool calls.
- **Outcome classes.** Took it at once means the next tool call after delivery reflects it; later means a following step; refused means the final file shows no change.
  Rationale: the three classes the task's table names.
- **Isolation.** Claude runs with `--settings <file> --setting-sources project --dangerously-skip-permissions` in a throwaway directory under `.scratch/`, with no `CLAUDE.md` mentioning `[creel]`, so trust comes only from the channel and the sentence.
  Rationale: the first spike's `CLAUDE.md` carried a creel rule; this one must test trust without it.
  Bound: bypass permissions apply only to the throwaway directory's sessions, and every strand is removed with `lym reed remove` after its capture.
- **Codex.** Hooks are passed with `-c hooks.PostToolUse=[…]` and the sentence through developer instructions; the cheapest model available to the CLI is chosen at run time and named in the report.
  Rationale: the first spike ran Codex this way.

## Constraints

- The Agent tool, Python and background gate runs are unavailable; scripts are bash.
- Scratch files live under `.scratch/`, never a system temp directory.
- `PATTERN-lym-temp-root` and `PATTERN-pane-text-injection` hold: prompts are typed only through reed and shuttle guards, not by hand-typed tmux keys.
- Markdown follows semantic line breaks.

## Auto-mode assumptions

- A model name or flag not accepted by the installed CLI is replaced by the nearest accepted one and the substitution is recorded in the report.
- A case whose strand cannot start is recorded as not run, with the error, instead of retried indefinitely.

## Open risks

- One run per case is a sample of one; the report says so.
- The Codex hook-trust bypass flag writes a trust entry to `~/.codex/config.toml`; the report names it.
- A model may ignore the scope change for reasons unrelated to the sentence; the report quotes its words and draws no further conclusion.

## Acceptance criteria

- The report exists at the path above with one section per case holding the config, script, sentence, quoted model words, a pane capture and the operator-visible text.
- The report ends with a case-by-outcome table that also marks what the operator could see.
- Every case's strand is removed and no stray `.scratch` process remains.
- `lym loom validate-description` passes on the change description.
