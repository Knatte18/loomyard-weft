# Decision record — plan-write-respawn

## Goal

Two defects surfaced by run `operator-surface` (Plan-Write, trace `7b2a74344919ecc3`):

1. A respawned Plan-Write session starts from an empty plan directory.
   The fresh-spawn rotation moves the prior session's finished plan into `_lyx/plan/archive-<stamp>/`,
   and nothing tells the new session it exists, so it rewrites the plan from nothing.
2. The Claude engine's `PreToolUse` hook refuses the `Agent` tool, and the spawned session is never told.
   The Plan-Write agent lost five parallel explorer dispatches before it started real work.

After this task, a respawned Plan-Write session is told exactly where the prior plan went and is instructed to carry it forward,
and every session the Claude engine spawns is told which tools its hooks refuse.

## Scope

In:

- Plan-Write's fresh-spawn path: the rotation stays, and when it moves files, the new session's prompt names the archive directory and the moved files and instructs reuse.
- The `shedadapters` fresh-spawn preparation seam, widened so a preparation can amend the prompt of the session about to spawn.
- A new loom stencil holding the prior-plan block's text.
- `internal/shuttleengine/claudeengine`: a system-prompt notice announcing each standing tool deny the engine installs, on both the launch and the resume command lines.
- Docs: the package docs of the touched packages, `docs/overview.md`'s shuttle paragraph (it describes the guardrails),
  and `internal/loomengine/plan.go`'s file comment, whose sentence calling stale-plan rotation "the future loom phase machine's responsibility" is false and must point at the rotator.

Out:

- Skipping the agent entirely when a finished plan already exists (see Decisions: the agent always runs).
- Discussion-Write's and Describe's own respawn behaviour (`archiveStaleOutputs` on their output files).
  The board task names Plan-Write only.
- The roadmap's Someday item "shuttle `Spec`: generic tools-restriction".
  This task announces the restrictions that already exist; it adds no new per-Spec restriction mechanism, and the roadmap does not move.
- Announcing the webster fork-context guard (`lyx webster` refused inside forks).
- Carrying the prior session's stop reason (its last message or Stuck reason) into the respawn prompt.

## Decisions

### D1 — The rotation stays; the respawn prompt points at the archive

Decision: Plan-Write keeps rotating stale top-level `.md` files into a fresh `archive-<stamp>[-N]` subdirectory before a fresh spawn.
When that rotation moves at least one file, the spawned session's prompt carries a prior-plan block naming the archive directory's absolute path and the moved file names.
When the rotation moves nothing (absent plan directory, or no top-level `.md` file), the prompt carries no block.

Rationale: shuttle's file contract requires `00-overview.md` to be absent at spawn, or the first turn end classifies Done on a stale file, so something must move it.
Rotating the whole plan keeps the directory in the one-plan-at-a-time state every downstream reader assumes.
Naming the exact directory the rotation just produced tells the session about this run's prior plan and no older archive, and costs nothing on a first spawn.

### D2 — The prior-plan block instructs carry-forward, not a rewrite

Decision: the block tells the session to:

- read the archived plan before writing anything;
- check it against the decision record and the current tree;
- carry forward every card that still holds by copying it into the plan directory rather than re-deriving it;
- re-verify every glyph it carries forward with `lyx quarry`;
- revise only what is wrong, missing or stale;
- never modify the archive directory;
- never copy `00-overview.md` from the archive: write it fresh, last, with `approved: false`, as the template already requires.

Rationale: the agent is the only party that can judge whether the prior plan still matches the decision record,
and a finished, still-valid plan then costs one verification pass instead of a full rewrite.
A copied `00-overview.md` would land before the revised cards and carry the archived plan's `approved` flag, which may be `true` on a restarted run.

### D3 — The agent always runs on a respawn

Decision: a respawn never short-circuits to Done on an existing plan, even one that passes the plan gate.

Rationale: a respawn follows a halt, and the halt's cause (an asking stop, a gate failure, an operator edit to the decision record) may invalidate the plan in ways the gate cannot see.
The session in D2 is the check that covers those cases.

### D4 — The fresh-spawn preparation can amend the spawned session's prompt

Decision: `shedadapters.SingleLLMProducer`'s fresh-spawn preparation hook is widened so it can append text to the end of the prompt of the session it prepares for.
The amendment is append-only: a preparation never rewrites or removes composed prompt text.
The producer applies the amendment to the spec it hands to the shuttle seam's run call, and to no other spec.
The attach path is unchanged: it never runs the preparation, and attach matching does not read the prompt.
A preparation that appends nothing leaves the prompt byte-identical.
The generic SingleLLM rows keep passing no preparation.

Rationale: the archive path exists only after the rotation runs, and the rotation must run after the attach probe (the probe-before-archive ordering in `SingleLLMProducer.Call`).
The spec is composed before the probe, so only the preparation step knows the path at the moment the prompt can still change.

### D5 — The prior-plan block is a stencil, appended to the prompt

Decision: the block's text lives in a new stencil under `contracts/stencils/loom/`, embedded and seeded like the existing loom stencils, with the archive path and the file list as its markers.
The Plan-Write rotator renders it and appends it to the end of the prompt.
`loom-template-plan.md` gains one static line in Step 3 telling the agent that a `Prior plan` section at the end of the prompt, when present, must be read before writing.

Rationale: agent-facing prose belongs in stencils, never in Go string literals.
The loom template carries no conditionals by rule, so the block is appended rather than rendered through an optional marker that `PlanSpec` would have to know about at compose time.

### D6 — The Claude engine announces every standing tool deny through `--append-system-prompt`

Decision: `claudeengine` passes `--append-system-prompt <notice>` on both the launch and the resume command line whenever it installs at least one announced deny, and omits the flag when it installs none.
The notice is built from the same inputs that build the `PreToolUse` hooks (the shuttle config, `interactive`, `forkSubagents`), with one sentence per installed deny.
Each sentence must hold in every session that receives it, forks included, since forks inherit the system prompt:

- Agent deny, non-fork run: the `Agent` tool is unavailable in this session; do all exploration and work in this session, with no subagents.
- Agent deny, fork run: the `Agent` tool accepts only fork subagents and refuses every other subagent type; a fork does its own work and never spawns further subagents.
- AskUserQuestion deny (autonomous runs with the deny on): `AskUserQuestion` is unavailable; when blocked or needing operator input, state the question as the final message and end the turn without writing the result file.

The webster fork-context guard is not announced.

Rationale: the engine is the one place that knows which denies it installs, so announcing there keeps the notice and the hooks from drifting when `claude_deny_agent_tool` or `claude_deny_ask_user_question` is switched off.
It covers every session `claudeengine` spawns rather than Plan-Write alone.
The notice states each steer in full because the deny reason reaches a session only on a refused call, which is the loss the notice exists to prevent.
A system-prompt flag is engine vocabulary, so shuttle's core stays a transport that never templates prompt content.
The resume line must carry the flag too: Claude Code replays the recorded system prompt on resume, but re-renders it from the current flags after a compaction and where recording is not enabled, so a resume line without it loses the notice.
A notice change therefore never reaches a running session, which matches the hooks, since those do not change mid-session either.
The webster guard needs no announcement: webster's fork stencils (`webster-prefix-fork.md`, `webster-template-integration.md`) already tell forks never to run `lyx webster`, and the guard is a backstop for a case the prompts cover.

### D7 — Notice text follows the steer strings' quoting constraint

Decision: the notice sentences are Go constants beside the existing steer constants in `settings.go`.
The assembled notice is a single line, its sentences joined by a space, and carries no line break, single quote, double quote or backslash, checked the same way the steer strings are checked.

Rationale: the notice is passed through `shell.Shell` quoting as one command-line argument, and reed types the whole command into the pane with `tmux send-keys`, which submits at the first line break.
Windows native-argument passing from pwsh can mangle double quotes and backslashes.
Both `Quote` implementations escape a single quote, and backticks and `$` are literal inside their single quotes;
the single quote stays forbidden for parity with the steer check.

## Constraints

- Probe-before-archive ordering in `SingleLLMProducer.Call` is preserved: the preparation, and therefore the prompt amendment, runs only after the attach probe finds no live run.
- Told-Geometry Invariant: `loomshed` and `shedadapters` derive no path; the plan directory stays resolved through `planparser.PlanDir(anchorPath)`, and the stencils directory reaches the rotator told, through the existing `shedrecipe` wiring.
- Lyxdirs Single-Declarer Invariant: no new `_lyx`/`.lyx` literal in path-construction context.
- The loom template stays conditional-free; the new stencil's markers are all required.
- Stencil vocabulary is shipped as an embedded default and seeded; the new stencil must be picked up by whatever enumeration the stencil tests pin.
- `shuttleengine.Spec` gains no field; the notice is engine-internal.
- `internal/shuttleengine/claudeengine` keeps the Treadle and Shed seam allowlists of its importers intact (no new imports into seam-bound packages).
- Semantic line breaks in every touched `.md` file.

## Auto-mode assumptions

- No operator answered questions; every pick in `## Decisions` is a self-pick recorded in the support log's question ledger.
- Claude Code's `--append-system-prompt` works in an interactive session (verified against `claude --help` on 2.1.285, which lists it and states the appended prompt is recorded on the first request and replayed on resume).
- `shuttleengine.Runner.AttachGated` matches candidates on output files only, never on the prompt, so amending the prompt after the probe cannot change what the probe would have matched.

## Open risks

- The first spawn of a restarted run whose plan directory holds a plan from an unrelated earlier attempt will be told to reuse it; D2's check against the decision record is the guard, and it relies on the agent following it.

## Acceptance criteria

- `loomshed` rotator, prior files present: the rotation moves every top-level `.md` file into a new `archive-<stamp>` directory, and the prompt amendment it produces names that directory's absolute path and every moved file name.
- `loomshed` rotator, absent plan directory and plan directory without top-level `.md` files: nothing moves, nothing is created, and the prompt is byte-identical to the composed one.
- `loomshed` rotator, same-second collision: the amendment names the `-N`-suffixed directory the files moved to, not the colliding one.
- `shedadapters.SingleLLMProducer`, respawn branch: the spec reaching the shuttle seam's run call carries the preparation's amendment; the attach-found branch never runs the preparation and its spec is unamended; a preparation error aborts before any spawn, as today.
- `shedadapters.SingleLLMProducer`, nil preparation: the run call's prompt is byte-identical to the spec source's.
- The new stencil is embedded, seeded, and read through `stencilstore`; the stencil enumeration tests include it; rendering with an empty marker fails loudly.
- `loom-template-plan.md` tells the agent to read a trailing `Prior plan` section before Step 3's writing.
- `claudeengine`, for every combination of `ClaudeDenyAgentTool`, `ClaudeDenyAskUserQuestion`, `interactive` and `forkSubagents`: the launch and resume commands carry an `--append-system-prompt` notice sentence for a deny if and only if `buildSettings` installs that deny's `PreToolUse` entry; the flag is absent when no announced deny is installed.
- `claudeengine`: the fork-run notice names fork subagents as the permitted type and tells a fork not to spawn further subagents; the non-fork notice names the `Agent` tool as unavailable; the AskUserQuestion notice states the final-message channel; the webster fork guard is never announced.
- `claudeengine`: every assembled notice is free of line breaks, single quotes, double quotes and backslashes, and a command built on each supported shell quotes the notice as one argument.
- `docs/overview.md`'s shuttle paragraph states that the engine announces its tool denies to the session; the package docs of `shedadapters`, `loomshed` and `claudeengine` describe the new behaviour; `internal/loomengine/plan.go`'s file comment names the rotator as owning stale-plan rotation.
- `go build ./...` and `go test ./...` pass.

## Notes for the plan writer

- The rotation is `NewPlanDirRotator`/`rotateStalePlanDir` in `internal/loomshed/planwrite.go`; it is wired in `internal/shedrecipe/entries_planwrite.go`, and the only other caller of `NewSingleLLMProducer` (`internal/shedrecipe/entries_singlellm.go`) passes a nil preparation.
- The deny hooks and steer strings are in `internal/shuttleengine/claudeengine/settings.go`; the launch and resume lines are `buildLaunchCmd`/`buildResumeCmd` in `command.go`.
