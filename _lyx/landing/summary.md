# Run gates in the foreground so a fork wakes Merriam once

A Webster batch fork used to start `lyx gate test` as a background Bash call and end its turn while it waited.
Claude Code reported the fork finished at each such stop, and every notice cost Merriam a turn that did nothing.
Quick checks run as background calls, `cd` into the working directory and system-temp output paths added more noise.

After this change every lyx-spawned agent and its forks run `lyx gate test` and quick checks in the foreground, inside their turn, with short commands.
Merriam wakes once per fork, for its final report.

No issue is closed by this change.

## What changes

- `internal/shuttleengine/claudeengine`: launch and resume lines carry `BASH_MAX_TIMEOUT_MS=3600000`, from one constant beside `CLAUDE_CODE_PROMPT_CACHE_TTL`.
  `steerRawGoDeny` no longer says "as a background Bash call"; it tells the session to run `lyx gate test` in the foreground with the Bash `timeout` at that ceiling, built from the same constant.
  It also says to run every other check in the foreground and not to `cd` into the session's own working directory.
- The implementer and verify-fix stencils forbid `run_in_background`, run gates and a card's `verify:` lines in the foreground, say the working directory already is the worktree root, and use worktree-relative paths in Bash commands.
  They point at the gate rule in the system prompt instead of restating the timeout.
- The Master template reads an interim fork notice ("still waiting on a command") as no completion and ends Merriam's turn with no tool call.
  The fixer path gets the same line, and a gate message that arrives while an earlier fixer has sent no final reply spawns no second fixer.
  Its sentence on a turn end with no background work and no outcome file is corrected: shuttle holds that turn end and notifies the parent, it does not end the run.
- The orch role stencil tells the orch to copy the gate rule from its own system prompt into the brief of a non-fork agent that runs `lyx gate test`.
- The plan stencil, `loom-plan-spec.md`, the `planparser` `verify-module-wide` refusal and `CLAUDE.md`'s push rule name the foreground route.
  No text outside `_lyx/` and test fixtures tells an agent to background `lyx gate test`.
- Package docs (`claudeengine`, `websterengine`, `shuttleengine` activity contract) and the `claims_test.go` rows are updated to match.

## Design decisions

- The steer is the one statement of the gate rule.
  It reaches every session through `--append-system-prompt` and a fork shares its parent's prompt, so stencils point at it.
  The orch brief copy for non-fork agents is the one sanctioned restatement, because a fresh agent does not see the orch's appended prompt.
- Turn-end activity now counts forks.
  Without interim notices a Merriam waiting on a working fork would read quiet for the whole batch, and the step idle kill (`step_idle_timeout_min`) or the quiet notice (`notice_quiet_min`) would fire.
  The reading takes the newest write across the main transcript and the `subagents/` transcripts, and reads a fork with an unmatched `tool_use` as active until the ceiling has passed since its last write.
  It stays inside `claudeengine`; `ReadAgentActivity` and its consumers change no code, only their documented contract.
- No new PATTERN entry: the rule is agent behavior pinned by the claims and claudeengine tests, not a code-structure invariant.

## To check by hand

Not run by this description; the gates `lyx gate test ./...` and `lyx gate test --tags integration ./...` are the push precondition.

- Confirm Claude Code honours `BASH_MAX_TIMEOUT_MS` as the ceiling on the Bash `timeout`.
  The first Webster run after landing shows it: more than one notice per fork means a gate ended or backgrounded at the default ten-minute ceiling.
  In that case a fixer whose gate runs long can spend `verify_gate_attempts` within a few Merriam turns and be removed mid-edit with Merriam's strand.
- Accepted fallback: a Bash call the tool still backgrounds at the one-hour ceiling sends an interim notice, and the Master template's reading of that notice rests on Merriam's reading of model-written text.
  A final notice misread as interim holds the batch unrecorded until `master_timeout_min` or the step idle kill.
- A fork stopped mid-call leaves a pending tool call that reads active for at most one hour past its last write.
- A gate that waits longer than the subagent prompt-cache TTL resumes the fork on a cold cache, as a backgrounded gate's completion notice does today.
