# Decision record — fork-wait-noise

## Goal

A Webster batch fork wakes Merriam once, with its final report, instead of once per gate wait.
Every lyx-spawned agent and its forks run `lyx gate test` and quick checks in the foreground, inside their turn, with short commands: no `cd` into the directory they already work in and no system-temp output paths.

## Scope

In:

- `internal/shuttleengine/claudeengine`: the launch and resume lines raise Claude Code's Bash timeout ceiling, the raw-go steer sentence names the foreground route with that ceiling, and the transcript activity reading counts a session's forks.
- Webster stencils: the implementer body, the verify-fix body and the Master template, including its sentence on a turn end with no background work and no outcome file (D5).
- The loom plan stencil, the loom plan spec, the `planparser` `verify-module-wide` refusal and `CLAUDE.md`'s push rule, which name `lyx gate test` "run as a background Bash call".
- The orch role stencil: its line on test runs going to a fork or an agent.
- `contracts/stencils/claims_test.go` rows and the claudeengine tests that pin the changed text and lines.
- Package docs: `internal/shuttleengine/claudeengine/doc.go`, `internal/websterengine/doc.go` where it describes how forks run gates, and the activity contract in `internal/shuttleengine/activity.go` and `internal/shuttleengine/doc.go` (D9).

Out:

- Master's backgrounded `recover-batch` and its backgrounded fork spawn: those turn ends are Merriam's intended waits and stay.
- Shuttle's background-shell wait machinery (`background_shell_wait_min`, Master's awaited shell prefix): unchanged.
- The subagent prompt-cache TTL bucket (`CLAUDE_CODE_SUBAGENT_PROMPT_CACHE_TTL`): unchanged.
- Any new gate verb, gate result record or await verb.

## Decisions

### D1 — Gates run in the foreground under a raised Bash timeout ceiling

Decision: every lyx-spawned claude session's launch and resume lines carry `BASH_MAX_TIMEOUT_MS=3600000`, set in `internal/shuttleengine/claudeengine` beside `CLAUDE_CODE_PROMPT_CACHE_TTL`, from one constant there.
An agent or fork runs each `lyx gate test` step as a foreground Bash call with the Bash tool's `timeout` at that ceiling, so the call returns inside the turn and the turn never ends while a gate runs.
`BASH_DEFAULT_TIMEOUT_MS` is left at Claude Code's default.

Bound: only a call that sets an explicit `timeout` runs past Claude Code's default, and no call stays in the foreground past one hour.
At the ceiling the Bash tool either ends the call or moves it to the background.
An ended gate is bounded by `lyx gate test` itself, which kills its go test tree and releases its slot on a terminating signal.
A backgrounded gate keeps running and holds its slot until it exits, as a backgrounded gate does today, and its completion notice is the interim notice D5 bounds.
The ceiling applies to every lyx-spawned session alike, the orch included, and to the forks those sessions spawn, since a fork runs in its parent's process.

Rationale: the fork ends its turn only because the gate is backgrounded; a foreground gate removes the cause.
The longest gate is the slot wait (`cli_wait_sec`, 300 s in the template) plus the build and the tests, which fits an hour; the default 10-minute ceiling does not cover a slot wait plus a tagged run.
A pane-line variable matches how the engine already carries Claude Code settings that the reed server's scrubbed environment cannot (PATTERN-shuttle-provider-seam keeps the Claude Code name inside claudeengine).

### D2 — The raw-go steer is the one statement of the gate rule

Decision: `steerRawGoDeny` drops "as a background Bash call" and says to run `lyx gate test` in the foreground with the Bash tool's `timeout` at the ceiling, the number built from the D1 constant rather than spelled twice.
It also says to run every other check in the foreground and not to `cd` into the session's own working directory.
Stencils that tell an agent how to run a gate or a quick check, or where to run a command from, point at "the gate rule in your system prompt" instead of restating the timeout value, the quick-check rule or the `cd` rule.

Rationale: the steer reaches every lyx-spawned session through `--append-system-prompt`, and a fork shares its parent's system prompt, so one sentence covers Webster's forks, recovery sessions, Burler and the orch's own forks.
Building the number from the constant keeps the steer and the pane line from drifting.

### D3 — The implementer and verify-fix bodies forbid backgrounding

Decision: `webster-body-implementer.md` replaces each "as its own Bash call with `run_in_background`, waited for" with a foreground call under the gate rule's timeout, for the card gate steps, a card's own `verify:` lines and the batch gate.
The body adds one rule: the fork never uses `run_in_background`, runs every other check as the gate rule says, and ends its turn only after writing the report.
`webster-body-verify-fix.md` gets the same change for its `lyx gate test --tags integration` step and the same rule, with the fixer ending its turn only after its final reply.

Rationale: a fork has no work it needs to background, so a flat ban is simpler than a rule about which calls may be backgrounded, and it guarantees the fork's only turn end is its final one.

### D4 — Commands run from the session's cwd with relative paths

Decision: the implementer and verify-fix bodies stop saying "from `{{.worktree_root}}`" for gate steps and commits, and instead say the session's working directory already is the worktree root, point at the gate rule for the `cd` rule, and give worktree-relative paths in Bash commands.
Paths passed to the Read and Write tools stay absolute, since those tools require it; `{{.worktree_root}}`, the card pointers and `{{.report_path}}` keep their absolute rendering.

Rationale: "run from `<absolute path>`" is what invites `cd <absolute pair path> &&`; the gate steps `renderCardGates` emits are already relative.
Rendering is unchanged, so PATTERN-told-geometry is untouched.

### D5 — Master treats an interim fork notice as no completion

Decision: the Master template's loop step 4 adds that a fork notice whose message says the fork is still waiting on a command is not its completion: Merriam ends its turn with no tool call.
Every other fork notice still goes to `record-batch`.
The fixer fork's step 4 under "A gate failure" gets the same line: an interim notice from the fixer ends Merriam's turn without rewriting the outcome and summary files.
The same section adds that a gate message arriving while an earlier fixer has sent no final reply spawns no second fixer: Merriam ends its turn with no tool call.
The Master template's sentence that a turn end with no background work and no outcome file ends the run with nothing done is corrected to shuttle's behaviour: such a turn end is held, with a hold notice to Merriam's parent, until the next notice starts a turn, `master_timeout_min`, the liveness check or loom's step idle kill.

Premise: a fork that sent an interim notice is no longer in shuttle's outstanding list.
Claude Code reports the fork `finished` at each interim stop (the board entry's pane capture), the Stop payload lists only `running` tasks as outstanding, and claudeengine's transcript fallback counts a fork launch as matched by any later `<task-notification>` naming it.

Bound: this edge lets a notice skip `record-batch`, or the fixer's rewrite, on Merriam's reading of model-written text.
With D1–D3 an interim notice arises only from a Bash call the tool backgrounds at the one-hour ceiling, which D1 treats as a hang.
- Batch loop, interim notice: the outcome file is missing, so shuttle holds the turn end and sends Merriam's parent one hold notice per interim stop; the fork's final notice starts Merriam's next turn.
  The parent notice in this fallback is accepted.
- Batch loop, final notice misread as interim: the turn end is held the same way and the batch stays unrecorded until `master_timeout_min` or loom's step idle kill, whichever comes first; the parent's hold notice shows the stall.
- A fork that never sends a final notice is bounded the same way.
- Fixer, interim notice: the outcome and summary files exist from the earlier arrival, so shuttle reads Merriam's turn end as a gated arrival and runs the verify gate while the fixer works.
  On a tree holding the fixer's uncommitted edits the gate fails at once on the dirty paths without running verify, and each failure rewrites the verify-gate report, the fixer's input file, with those paths.
  Its re-prompt reaches Merriam, who spawns no second fixer and ends its turn, which is another arrival, so `verify_gate_attempts` is spent within a few Merriam turns, well before a mid-edit fixer's final notice.
  The spent budget finalizes the run stuck, and since Master's spec does not keep its pane, shuttle removes the strand with the fixer fork still in it, leaving the tree in whatever state the fixer reached.
  This outcome is accepted, not remedied: with D1 the trigger is a fixer call the tool backgrounds at the one-hour ceiling, which D1 treats as a hang, and two fixers never edit the tree at once.

Rationale: step 4 as written would send an interim notice to `record-batch`, which classifies the missing report as a failed batch while the fork still works; the fallback (a Bash call the tool still backgrounds) needs a safe reading.
Skipping the fixer's rewrite does not keep Merriam off the gate, since shuttle decides an arrival from the output files, so the fixer case is bounded by the attempt budget and the one-fixer rule instead.

### D6 — The orch's agent briefs point at the gate rule

Decision: the orch role stencil's line on test runs going to a fork or an agent adds that a brief for a non-fork agent that runs `lyx gate test` carries a copy of the gate rule, timeout value included, taken from the orch's own system prompt.
The stencil names where the orch copies it from and never spells the rule's content, so this runtime copy is the one sanctioned restatement outside the steer.

Rationale: an orch fork inherits the steer, but a fresh non-fork agent does not see the orch's appended system prompt, so the orch carries the rule into that agent's brief itself.

### D7 — Every "background Bash call" naming of the gate route changes

Decision: `loom-template-plan.md`, `loom-plan-spec.md` and the `planparser` `verify-module-wide` refusal stop saying "run as a background Bash call" and name `lyx gate test [-C <module>] [--tags <tags>] <packages>` run in the foreground under the gate rule.
`CLAUDE.md`'s push rule drops "each run as its own background Bash call" and says each tier runs as its own Bash call as the gate rule in the session's system prompt says.
No text left in the repo outside `_lyx/` and test fixtures tells an agent to background `lyx gate test`.

Rationale: a single leftover instruction re-creates the wake pattern for whoever reads it.

### D8 — No new PATTERN entry

Decision: the foreground-gate rule is pinned by the claims tests and the claudeengine tests, not by a PATTERN entry.

Rationale: it is a rule for agent behavior stated in one steer sentence, not a code-structure invariant a scan could hold; the claims tests already guard stencil wording this way.

### D9 — A session's forks count as its activity

Decision: claudeengine's turn-end activity reading takes the newest write across the session's main transcript and the fork transcripts under its `subagents/` directory, and reads a fork whose transcript holds a `tool_use` with no matching `tool_result` as active at the time of reading, wherever that call sits in the transcript, so a held gate followed by a parallel sibling call's result still reads active.
The reading stays inside claudeengine, so `ReadAgentActivity` and its consumers, loom's step idle kill and batten's quiet notice, change no code.
Their documented contract changes with it: `TurnEndActivity.TranscriptModTime` and `AgentActivity.LastActivity` in `internal/shuttleengine/activity.go`, and the activity paragraph in `internal/shuttleengine/doc.go`, say the session's last activity, its forks included, rather than the transcript's last write.

A pending tool call reads active only until D1's ceiling has passed since the fork transcript's last write, and after that the fork reads that write time, so the reading needs no proof the call is still held.

Bound: a fork stopped or interrupted mid-call leaves a pending tool call with no writer, and it reads active for at most D1's ceiling (one hour) past its last write, after which the idle kill and the quiet notice see it as quiet.
A live call ends or backgrounds at the same ceiling, so the cap never cuts a held call short; a fork with no call in flight reads only its last write, as a main transcript does today.

Rationale: activity is read from the main transcript's modification time and the newest Stop line, and a fork writes neither, so with interim notices gone a Merriam waiting on a working fork reads quiet for the whole batch; `step_idle_timeout_min` (60) would kill such a step and `notice_quiet_min` (30) would notice it.
Lowering the ceiling below the idle window does not help, since the quiet stretch is the whole batch, not one call.

## Constraints

- PATTERN-shuttle-provider-seam: `BASH_MAX_TIMEOUT_MS` and the Bash tool's `timeout` semantics are named in Go only under `internal/shuttleengine/claudeengine`; the webster and loom packages never import it.
- PATTERN-stencil-ownership: stencils stay files read at call time; the steer stays a Go constant in claudeengine as today.
- PATTERN-producer-pointer-rule: stencils point at the system-prompt gate rule for the timeout value rather than repeating it.
- PATTERN-no-denied-recovery: the new way forward in the `verify-module-wide` refusal names a command the agents' settings allow.
- PATTERN-refusal-way-forward: the refusal's spec row in `loom-plan-spec.md` changes in the same commit as the refusal text.
- PATTERN-comment-line-breaks and semantic line breaks in every edited `.md` file.
- PATTERN-test-economy: extend the existing launch-line, steer and claims tests; add no new top-level test unless no existing one fits.
- Docs land in the same commit: claudeengine `doc.go` names the new pane-line variable and its bound and says the activity reading counts forks; shuttleengine's activity contract says the session's last activity, forks included (D9); websterengine `doc.go` describes forks running gates in the foreground.

## Auto-mode assumptions

- Claude Code honours `BASH_MAX_TIMEOUT_MS` as the ceiling on the Bash tool's `timeout`, and an explicit `timeout` above two minutes keeps a call in the foreground; the Burler fixer's 300000 ms `lyx shuttle await` calls already run in the foreground in production.
- A fork's Bash working directory is the worktree root, because Master runs in it and forks inherit it.
- An Agent-tool fork shares its parent's appended system prompt and process environment; a non-fork subagent does not see the appended system prompt.
- One hour covers every gate an agent runs, the orch's pre-push module-wide tiers included.
- A fork writes its own transcript under the session's `subagents/` directory, never the main transcript, and an in-flight tool call shows there as a tool-use entry with no result yet; claudeengine's usage reading and fork audit already read that directory.

## Open risks

- If Claude Code ignores `BASH_MAX_TIMEOUT_MS`, a gate over ten minutes still ends or backgrounds at the default ceiling; D5's bounds then apply to every such gate, so the parent gets a hold notice per interim stop, and a fixer whose gate runs over ten minutes ends the run stuck within a few Merriam turns and is removed mid-edit with Merriam's strand, as D5's fixer bound describes; the first Webster run after landing shows it as more than one notice per fork.
- A gate that waits longer than the subagent prompt-cache TTL resumes the fork on a cold cache, as a backgrounded gate's completion notice does today; this design neither improves nor worsens that.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - fixer-fork-notice-path-unguarded (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - foreground-waits-meet-idle-watchdogs (class design, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Discussion-Review -->
## Acceptance criteria

- Launch and resume lines of every role carry `BASH_MAX_TIMEOUT_MS=3600000`, and the value comes from one claudeengine constant.
- The appended deny notice's raw-go sentence names the foreground route with the Bash `timeout` at that value, quick checks in the foreground and no `cd` into the working directory, and no longer says "background Bash call".
- The rendered implementer prompt names no `run_in_background`, says the fork never backgrounds a call, runs card gate steps, card `verify:` lines and the batch gate in the foreground under the gate rule, says the working directory is the worktree root, points at the gate rule for `cd` and quick checks, and asks for worktree-relative paths in Bash commands.
- The rendered verify-fix prompt carries the same no-backgrounding rule, runs its integration gate in the foreground under the gate rule, says the working directory is the worktree root, points at the gate rule for `cd` and quick checks, and asks for worktree-relative paths in Bash commands.
- The Master template's loop step 4 and its fixer-fork step 4 say an interim fork notice that says the fork is still waiting ends Merriam's turn with no tool call.
- The Master template's gate-failure section says a gate message arriving while an earlier fixer has sent no final reply spawns no second fixer, and its sentence on a turn end with no background work and no outcome file says the turn end is held with a notice to the parent, not that the run ends.
- The orch role stencil tells the orch to copy the gate rule from its own system prompt into a test-running non-fork agent's brief.
- The activity reading of a session whose fork holds a tool call open, or wrote its transcript more recently than the main transcript, reports that fork's activity.
- A fork whose pending tool call was last written more than the Bash timeout ceiling ago reads that write time, not active.
- The `verify-module-wide` refusal, the plan stencil, the plan spec and `CLAUDE.md` name the foreground route; `git grep "background Bash call"` finds no text outside `_lyx/` about `lyx gate test`.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` are green.

## Notes for the plan writer

- Text naming the background route today: `internal/shuttleengine/claudeengine/settings.go` (`steerRawGoDeny`), `internal/planparser/verifymodule.go`, `contracts/stencils/loom/loom-template-plan.md`, `contracts/specs/loom-plan-spec.md`, `contracts/stencils/claims_test.go`, `contracts/stencils/webster/webster-body-implementer.md`, `contracts/stencils/webster/webster-body-verify-fix.md`, `CLAUDE.md`.
- The pane-line variables are assembled in `buildLaunchCmd` and `buildResumeCmd` in `internal/shuttleengine/claudeengine/command.go`.
