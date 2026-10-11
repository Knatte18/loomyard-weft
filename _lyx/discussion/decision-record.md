# Decision record — fork-wait-noise

## Goal

A Webster batch fork wakes Merriam once, with its final report, instead of once per gate wait.
Every lyx-spawned agent and its forks run `lyx gate test` and quick checks in the foreground, inside their turn, with short commands: no `cd` into the directory they already work in and no system-temp output paths.

## Scope

In:

- `internal/shuttleengine/claudeengine`: the launch and resume lines raise Claude Code's Bash timeout ceiling, and the raw-go steer sentence names the foreground route with that ceiling.
- Webster stencils: the implementer body, the verify-fix body and the Master template.
- The loom plan stencil, the loom plan spec and the `planparser` `verify-module-wide` refusal, which name `lyx gate test` "run as a background Bash call".
- The orch role stencil: its line on test runs going to a fork or an agent.
- `contracts/stencils/claims_test.go` rows and the claudeengine tests that pin the changed text and lines.
- Package docs: `internal/shuttleengine/claudeengine/doc.go`, `internal/websterengine/doc.go` where it describes how forks run gates.

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

Bound: only a call that sets an explicit `timeout` runs past Claude Code's default; no foreground call outlives one hour, after which the Bash tool ends it, and `lyx gate test` already kills its go test tree and releases its slot on a terminating signal.
The ceiling applies to every lyx-spawned session alike, the orch included, and to the forks those sessions spawn, since a fork runs in its parent's process.

Rationale: the fork ends its turn only because the gate is backgrounded; a foreground gate removes the cause.
The longest gate is the slot wait (`cli_wait_sec`, 300 s in the template) plus the build and the tests, which fits an hour; the default 10-minute ceiling does not cover a slot wait plus a tagged run.
A pane-line variable matches how the engine already carries Claude Code settings that the reed server's scrubbed environment cannot (PATTERN-shuttle-provider-seam keeps the Claude Code name inside claudeengine).

### D2 — The raw-go steer is the one statement of the gate rule

Decision: `steerRawGoDeny` drops "as a background Bash call" and says to run `lyx gate test` in the foreground with the Bash tool's `timeout` at the ceiling, the number built from the D1 constant rather than spelled twice.
It also says to run every other check in the foreground and not to `cd` into the session's own working directory.
Stencils that tell an agent how to run a gate point at "the gate rule in your system prompt" for the timeout value instead of restating the number.

Rationale: the steer reaches every lyx-spawned session through `--append-system-prompt`, and a fork shares its parent's system prompt, so one sentence covers Webster's forks, recovery sessions, Burler and the orch's own forks.
Building the number from the constant keeps the steer and the pane line from drifting.

### D3 — The implementer and verify-fix bodies forbid backgrounding

Decision: `webster-body-implementer.md` replaces each "as its own Bash call with `run_in_background`, waited for" with a foreground call under the gate rule's timeout, for the card gate steps, a card's own `verify:` lines and the batch gate.
The body adds one rule: the fork never uses `run_in_background`, runs quick checks (`lyx loom lint-comments`, `lyx loom lint-pattern`, `lyx quarry resolve`, reads) in the foreground at the default timeout, and ends its turn only after writing the report.
`webster-body-verify-fix.md` gets the same change for its `lyx gate test --tags integration` step.

Rationale: a fork has no work it needs to background, so a flat ban is simpler than a rule about which calls may be backgrounded, and it guarantees the fork's only turn end is its final one.

### D4 — Commands run from the session's cwd with relative paths

Decision: the implementer and verify-fix bodies stop saying "from `{{.worktree_root}}`" for gate steps and commits, and instead say the session's working directory already is the worktree root: never `cd`, and give worktree-relative paths in Bash commands.
Paths passed to the Read and Write tools stay absolute, since those tools require it; `{{.worktree_root}}`, the card pointers and `{{.report_path}}` keep their absolute rendering.

Rationale: "run from `<absolute path>`" is what invites `cd <absolute pair path> &&`; the gate steps `renderCardGates` emits are already relative.
Rendering is unchanged, so PATTERN-told-geometry is untouched.

### D5 — Master treats an interim fork notice as no completion

Decision: the Master template's loop step 4 adds that a fork notice whose message says the fork is still waiting on a command is not its completion: Merriam ends its turn with no tool call.
Every other fork notice still goes to `record-batch`.

Rationale: with D1–D3 an interim notice becomes rare, but step 4 as written would send one to `record-batch`, which classifies the missing report as a failed batch while the fork still works; the fallback (a Bash call the tool still backgrounds) needs a safe reading.

### D6 — The orch's agent briefs point at the gate rule

Decision: the orch role stencil's line on test runs going to a fork or an agent adds that a brief for an agent that runs `lyx gate test` tells it to run it as the gate rule in the orch's own system prompt says.
No other orch text restates the rule.

Rationale: an orch fork inherits the steer, but a fresh non-fork agent does not see the orch's appended system prompt, so the orch carries the rule into that agent's brief by pointing at the steer it can read.

### D7 — Every "background Bash call" naming of the gate route changes

Decision: `loom-template-plan.md`, `loom-plan-spec.md` and the `planparser` `verify-module-wide` refusal stop saying "run as a background Bash call" and name `lyx gate test [-C <module>] [--tags <tags>] <packages>` run in the foreground under the gate rule.
No production text left in the repo tells an agent to background `lyx gate test`.

Rationale: a single leftover instruction re-creates the wake pattern for whoever reads it.

### D8 — No new PATTERN entry

Decision: the foreground-gate rule is pinned by the claims tests and the claudeengine tests, not by a PATTERN entry.

Rationale: it is a rule for agent behavior stated in one steer sentence, not a code-structure invariant a scan could hold; the claims tests already guard stencil wording this way.

## Constraints

- PATTERN-shuttle-provider-seam: `BASH_MAX_TIMEOUT_MS` and the Bash tool's `timeout` semantics are named in Go only under `internal/shuttleengine/claudeengine`; the webster and loom packages never import it.
- PATTERN-stencil-ownership: stencils stay files read at call time; the steer stays a Go constant in claudeengine as today.
- PATTERN-producer-pointer-rule: stencils point at the system-prompt gate rule for the timeout value rather than repeating it.
- PATTERN-no-denied-recovery: the new way forward in the `verify-module-wide` refusal names a command the agents' settings allow.
- PATTERN-refusal-way-forward: the refusal's spec row in `loom-plan-spec.md` changes in the same commit as the refusal text.
- PATTERN-comment-line-breaks and semantic line breaks in every edited `.md` file.
- PATTERN-test-economy: extend the existing launch-line, steer and claims tests; add no new top-level test unless no existing one fits.
- Docs land in the same commit: claudeengine `doc.go` names the new pane-line variable and its bound; websterengine `doc.go` describes forks running gates in the foreground.

## Auto-mode assumptions

- Claude Code honours `BASH_MAX_TIMEOUT_MS` as the ceiling on the Bash tool's `timeout`, and an explicit `timeout` above two minutes keeps a call in the foreground; the Burler fixer's 300000 ms `lyx shuttle await` calls already run in the foreground in production.
- A fork's Bash working directory is the worktree root, because Master runs in it and forks inherit it.
- An Agent-tool fork shares its parent's appended system prompt and process environment; a non-fork subagent does not see the appended system prompt.
- One hour covers every gate an agent runs, the orch's pre-push module-wide tiers included.

## Open risks

- If Claude Code ignores `BASH_MAX_TIMEOUT_MS`, a gate over ten minutes still ends or backgrounds at the default ceiling; D5 keeps Merriam safe in that case, and the first Webster run after landing shows it as more than one notice per fork.
- A gate that waits longer than the subagent prompt-cache TTL resumes the fork on a cold cache, as a backgrounded gate's completion notice does today; this design neither improves nor worsens that.
- An activity reader that judges a session quiet from its transcript might read an hour-long foreground tool call as quiet; the Plan writer checks `shuttleengine`'s activity reading and batten's idle notices for a tool call held open that long.

## Acceptance criteria

- Launch and resume lines of every role carry `BASH_MAX_TIMEOUT_MS=3600000`, and the value comes from one claudeengine constant.
- The appended deny notice's raw-go sentence names the foreground route with the Bash `timeout` at that value, quick checks in the foreground and no `cd` into the working directory, and no longer says "background Bash call".
- The rendered implementer prompt names no `run_in_background`, says the fork never backgrounds a call, runs card gate steps, card `verify:` lines and the batch gate in the foreground under the gate rule, and tells the fork never to `cd`.
- The rendered verify-fix prompt runs its integration gate in the foreground under the gate rule.
- The Master template's loop says an interim fork notice that says the fork is still waiting ends Merriam's turn with no tool call.
- The orch role stencil tells the orch to point a test-running agent's brief at the gate rule.
- The `verify-module-wide` refusal, the plan stencil and the plan spec name the foreground route; `git grep "background Bash call"` finds no production text about `lyx gate test`.
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` are green.

## Notes for the plan writer

- Text naming the background route today: `internal/shuttleengine/claudeengine/settings.go` (`steerRawGoDeny`), `internal/planparser/verifymodule.go`, `contracts/stencils/loom/loom-template-plan.md`, `contracts/specs/loom-plan-spec.md`, `contracts/stencils/claims_test.go`, `contracts/stencils/webster/webster-body-implementer.md`, `contracts/stencils/webster/webster-body-verify-fix.md`.
- The pane-line variables are assembled in `buildLaunchCmd` and `buildResumeCmd` in `internal/shuttleengine/claudeengine/command.go`.
