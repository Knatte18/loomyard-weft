# Plan-Write respawn points at prior plan; engine announces tool denies

A respawned Plan-Write session used to start from an empty plan directory and rewrite the plan from nothing.
The fresh-spawn rotation moves the prior session's finished plan into `_lyx/plan/archive-<stamp>/`, and nothing told the new session it existed.
Separately, the Claude engine's `PreToolUse` hook refuses the `Agent` tool without the spawned session knowing, so a Plan-Write agent lost five parallel explorer dispatches before real work started (run `operator-surface`, trace `7b2a74344919ecc3`).

## What changes

- **Prior-plan block.**
  When the Plan-Write rotation moves at least one file, the new session's prompt ends with a `Prior plan` section.
  It names the archive directory's absolute path and the moved file names.
  It tells the session to read the archived plan, check it against the decision record and the current tree, copy over every card that still holds, and re-verify carried glyphs with `lyx quarry`.
  It also forbids modifying the archive and copying `00-overview.md`, which is written fresh and last with `approved: false`.
  When nothing moves, the prompt is byte-identical to the composed one.
  The text lives in the new `loom-template-prior-plan` stencil, and `loom-template-plan.md` gains one line telling the agent to read that section first.
- **Prompt amendment seam.**
  `shedadapters.SingleLLMProducer`'s fresh-spawn preparation can now append text to the prompt of the session it prepares for.
  The amendment applies only to the spec handed to the run call, after the attach probe finds no live run.
  The attach path never runs the preparation, and a nil preparation leaves the prompt untouched, so the generic SingleLLM rows are unchanged.
- **Deny notice.**
  `claudeengine` passes `--append-system-prompt <notice>` on both the launch and resume command lines, with one sentence per standing deny it installs.
  The flag is omitted when no announced deny is installed.
  The notice is built from the same inputs as the hooks, so switching off `claude_deny_agent_tool` or `claude_deny_ask_user_question` drops the matching sentence.
- **Docs.**
  The package docs of `claudeengine`, `loomshed` and `shedadapters`, the shuttle paragraph of `docs/overview.md`, and the file comment of `internal/loomengine/plan.go` are updated.
  That comment wrongly called stale-plan rotation the future phase machine's job and now names the rotator.

## Design decisions

- The rotation stays, because shuttle's file contract needs `00-overview.md` absent at spawn.
  The agent always runs on a respawn and never short-circuits to Done on an existing plan.
  The halt that caused the respawn may have invalidated the plan in ways the plan gate cannot see.
- The prior-plan block is appended to the prompt rather than rendered through an optional marker, since the loom template carries no conditionals.
- The deny sentences hold in every session that receives them, forks included, because forks inherit the system prompt.
  The `Agent` sentence differs for fork runs: only fork subagents are permitted, and a fork spawns none.
  The `AskUserQuestion` sentence tells the session to state its question as the final message and end the turn without writing the result file.
- The notice is one line with no line break, single quote, double quote or backslash, checked like the existing steer strings.
  `reed` types the command via `tmux send-keys`, which submits at the first line break, and pwsh can mangle double quotes and backslashes.
- The resume line carries the flag too.
  Claude Code re-renders the system prompt from the current flags after a compaction, so a resume line without it would lose the notice.
- `shuttleengine.Spec` gains no field, and the webster fork-context guard is not announced because webster's fork stencils already cover it.
- Discussion-Write and Describe respawn behaviour is untouched.

## To check by hand

- The restarted-run risk: a first spawn whose plan directory holds a plan from an unrelated earlier attempt is told to reuse it.
  The check against the decision record is the only guard, and it relies on the agent following it.
- A live respawn of Plan-Write that reads the `Prior plan` section and carries cards forward was not run.
- A live Claude session started with `--append-system-prompt` on a real interactive pane, launch and resume, was not run.
  The flag was verified only against `claude --help`.
