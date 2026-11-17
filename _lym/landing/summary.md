# Move Claude-only wording into engine blocks with Codex copies

Stencils, specs and the Go text an agent reads no longer name Claude Code tools outside the Claude engine's own places.
Each passage that did moves into an engine block with a Claude copy and a Codex copy.
A Codex session now gets the same instruction in Codex's terms: a fresh subagent is `spawn_agent` with `fork_turns: "none"`,
a fork is `spawn_agent` with `fork_turns: "all"`, waiting is `wait_agent`, an edit is `apply_patch`,
a read is a read-only shell command, "your system prompt" is "your developer instructions",
and an operator question in interactive mode is asked in the pane.
A scan holds the rule from here on.

## What changes

- New engine blocks under `contracts/stencils/engine/claude/` and `contracts/stencils/engine/codex/`, registered in `contracts/stencils/stencils.go`.
  They cover forking, waiting and messaging forks, subagent lookups, darn subagents, the question tool, absolute paths,
  foreground timeouts, standing instructions and backgrounded commands.
- The burler, darn, gatefix, loom, raddle, twill, weaver and webster stencils include those blocks instead of naming the tools.
  `webster-template-master.md` runs `recover-batch` and the fix-verify step through the `engine-background-command` block.
  The Claude copy keeps today's wording; the Codex copy runs the command in the foreground and acts on its exit and output.
- Go text an agent reads is chosen per engine or worded for any engine:
  `mode_rules` in `internal/loomengine`, the plan-gate finding messages in `internal/planparser`,
  and the text in `internal/websterengine`, `internal/darnengine`, `internal/driftengine`, `internal/burlerengine`,
  `internal/twilldirective`, `internal/gatefixer`, `internal/shellguard`, `internal/seatengine` and `internal/loomcli`.
- `internal/stencilstore` gains `ReadForEngine`, which reads a stencil with its engine blocks written in place.
  Producers that fill a stencil with no include map call it, so the text is already worded for the launch's engine.
  `ReadSet` now also writes in place the engine blocks that a non-engine include carries.
  An engine block copy is read without its file's final newline, so a block can stand inside a line.
  An engine-block include left unresolved is refused, with a way forward in the error.
- `PATTERN-stencil-ownership` states the rule: one engine's wording only in its engine blocks, rendered for the launch's engine.

## The scan

`contracts/stencils/engineterms_test.go` runs at `tier0` on the repository's files.
It holds a closed list of terms: the Agent tool, `subagent_type`, fork subagents, the Skill, Read, Edit, Write, Grep and Glob tools,
`AskUserQuestion`, "system prompt", and the Bash tool's `timeout` and `run_in_background`.
It fails on a term in `contracts/stencils/` or `contracts/specs/` outside `contracts/stencils/engine/claude/`,
and in a production Go string literal outside `internal/shuttleengine/claudeengine/`.
It fails on an engine block that a registered engine has no copy of.
It skips `contracts/stencils/orch/`, `contracts/orchrole/`, the orch packages and the test kits, since the orch stays on Claude.

## Design decisions

- A file tool counts as a term only when the text names it as a tool ("the Read tool", "with Write"),
  so an imperative such as "Write the report" does not trip the scan.
- A Go literal that never reaches a Codex session goes on an allowlist.
  Each key names the literal's file, its enclosing declaration and the term, and each entry carries a reason.
  The scan does not check a reason's claim; review does.
- The Codex copies were checked against `docs/research/codex/codex-twill-spike-2026-10-24/report.md` (codex-cli 0.164.0):
  `fork_turns: "all"` carries the parent's whole history to the child and takes no `model` or `reasoning_effort` override.

## To check by hand

- Each Codex engine block and each per-engine Go text says what its Claude copy says, in Codex's terms.
- `producer-engine-choice-untested` (design, medium): nothing pins that each producer passes the launch's engine to `ReadForEngine`;
  look at the producers in `internal/websterengine`, `internal/gatefixer`, `internal/twilldirective` and `internal/loomengine`.
- No regression check and no live Codex session ran against the new blocks.
