## Goal

Claude-only wording moves into Claude engine blocks, each with a Codex copy, held by a term scan

Takes: prompt-engine-per-launch's engine-aware renders, so an engine block resolves for the launch's engine; codex-standing-instructions' final heddle and codexengine text, which the scan then judges as landed
Delivers: every stencil and spec passage naming a closed-list term in a Claude engine block with a Codex copy; mode_rules and plan-gate finding text chosen per engine or worded for any; the D2 scan over stencils, specs and production Go literals, with orch places skipped, an allowlist with reasons for literals that never reach a Codex session, and a check that every engine block has a copy for every registered engine
Exports: nothing another part calls

## Scope

The part changes these packages:

- contracts/stencils (about 1400 changed lines)
- contracts/specs (about 150 changed lines)
- internal/loomengine (about 120 changed lines)
- internal/websterengine (about 80 changed lines)
- internal/driftengine (about 40 changed lines)
- internal/burlerengine (about 40 changed lines)
- internal/darnengine (about 20 changed lines)
- internal/planparser (about 40 changed lines)
- internal/batcher (about 20 changed lines)
- internal/shellguard (about 40 changed lines)
- internal/seatengine (about 20 changed lines)
- internal/loomcli (about 20 changed lines)

## Decisions

### D2 — Claude-only wording lives only in Claude engine blocks

Every stencil or spec passage that names a Claude Code tool or concept moves into an engine block with a Codex copy.
Prompt text Go fills in, such as the `mode_rules` value and plan-gate finding messages an agent reads, is chosen per engine or worded for any engine.
The Codex copy says the same thing in Codex's terms:
a fresh subagent is `spawn_agent` with `fork_turns: "none"`, a fork is `spawn_agent` with `fork_turns: "all"`, waiting is `wait_agent`, an edit is `apply_patch`, a read is a read-only shell command,
"your system prompt" is "your developer instructions", and a question for the operator in interactive mode is asked in the pane.

A scan holds the rule over a closed term list:
the Agent tool, `subagent_type`, fork subagents, the Skill, Read, Edit, Write, Grep and Glob tools, AskUserQuestion, "system prompt", and the Bash tool's `timeout` and `run_in_background`.
It fails on a term in `contracts/stencils/` or `contracts/specs/` outside `contracts/stencils/engine/claude/`, and in a production Go string literal outside `internal/shuttleengine/claudeengine/`.
It skips `contracts/stencils/orch/`, `contracts/orchrole/` and the orch packages, since the orch stays on Claude.
Bound: an allowlist entry with a reason admits a Go literal that never reaches a Codex session;
the reason's claim is held by review, not by the scan.
The scan also fails on an engine block some registered engine has no copy of.

Rationale: engine blocks exist for this (`internal/stencilstore/engineblock.go`), but only the assist seats use them.
Fact (`docs/research/codex/codex-twill-spike-2026-10-24/report.md`, codex-cli 0.164.0): `fork_turns: "all"` carries the parent's whole history to the child, and such a fork accepts no `model` or `reasoning_effort` override.

### Added after Discussion (arbiter, 2026-11-16): The assist-trial schedule risk has no subject

Decision: Drop the open risk on codex-assist-review-trial and close Q14 as answered: this task takes no scheduling call on that trial. The three runs the trial chose (operator, 2026-11-15), prime-is-root-beam, hub-stop-button and machine-restart-shortcut, no longer exist as tasks: the operator merged them into adopt-build-resplit (2026-11-16) to be split again from scratch, and that entry carries no code_assist_reviewer_model setting, so no run with a Codex assist seat is queued or running. Any runs the trial picks afresh are the trial note's own choice, so this record states no scheduling bound.

Rationale: Q14 asked whether the three chosen runs go before or after this task lands. Board facts: lym board find code_assist_reviewer_model matches only the trial note, and adopt-build-resplit's body says the beam's remaining members were merged into it. So there is nothing to schedule, and choosing future trial runs belongs to codex-assist-review-trial, not to this task's design.

### Added after Discussion (arbiter, 2026-11-16): Round-3 review findings are answered by the record as it stands

Decision: Treat the round-3 findings d12-non-exec-calls-outside-cell-model (F1), d1-scan-reaches-probe-literals (F2) and d12-read-again-path-resolution-unstated (F3) as answered by D1, D4 and D12 as written; the plan carries D12's function_call rules, D1's launch-spec literal definition (a shuttleengine.Spec literal that sets Prompt, attach probes skipped, inline prompts split into a renderer) and D12's per-call cwd resolution as stated, with no further change.

Rationale: The Discussion review reached its hard cap at round 3 with these keys open. D12 now scopes the payload and cell sentences to nested calls, gives a non-Bash, non-apply_patch payload no nudge and leaves the rollout unread for it, and ends a lookup_run or edit_run chain at a function_call; the 2026-11-15 rollout holds 21 custom_tool_call exec items and four collaboration function_call items (two spawn_agent, two send_message) as D12 states. D1 and D4 skip a Spec literal that sets no Prompt; bouncer.go's awaitLiveJudge and awaitLiveSeed literals set only OutputFiles, Role, Segment and Round, while judgeCall and runSeedSpawn set Prompt inline. D12 resolves both read paths against each call's CommandExecution cwd; the 2026-11-16 spike rollout carries that cwd as a file:// URL and parsed_cmd paths as written (notes.txt).

### Added after Discussion (parent, 2026-11-17): Master's backgrounded commands get an engine block

Decision: The fixer takes F2 of code round 2: the passages of webster-template-master.md that run recover-batch or the fix-verify step as a backgrounded Bash command and act on its completion notification move into an engine block (such as engine-background-command). The Claude copy keeps today's wording; the Codex copy tells a Codex session to run the command in the foreground through its shell tool and act on the command's exit and output when it returns. Both copies are registered in contracts/stencils/stencils.go.

Rationale: run_in_background is on the record's closed list of Claude-only terms, so these passages are inside the task's scope; a Codex session must get the same instructions as a Claude one in its own terms, and running in the foreground is how a Codex session learns that a command ended.

## Constraints

- `PATTERN-shuttle-provider-seam`: Claude's user-setup resolution, session scope and file locations stay in `claudeengine`, and reach `codexengine` only through `shuttleengine` interfaces wired in `internal/shuttleengines`; the rollout reading for the batching nudge stays in `codexengine`, and the read-only lookup check both engines' nudges use sits in its own package under `internal/shuttleengine`.
- `PATTERN-stencil-ownership`: engine text in engine blocks, resolved by `internal/stencilstore`'s one resolver; skills stay typed on the launch spec, and the skill listing is no request to load one.
  Its background's edit-directive section changes with D3: the engine copies are the directive's only statement.
- `PATTERN-heddle-instructions`: the shipped-entry changes and the new target live in `internal/heddleengine`; `HEDDLE.md` states the new target, and the background names the hub reconcile as a file-target render site.
- `PATTERN-child-processes`: the plugin hook runs are logged, waited on and registered as the pattern says.
- `PATTERN-tolerant-records`: the manifest record is written through `internal/state` and read tolerantly.
- `PATTERN-config-layers`: the new codex.yaml keys ship through `configengine` with their template defaults.
- `PATTERN-cli-cobra`: the verb joins shuttle's cobra subtree.
- `PATTERN-test-speed`: the parity test and the scans run at `tier0` on fixtures, and no test pins a stencil's prose.

## Auto-mode assumptions

- D1: no exemption table; every compared difference found has a Codex channel.
- D6: subagents run on the session's model; 16 sits above the widest fan any stencil asks for.
- D12: the nudge keeps separate settings per engine rather than moving to a shared module, which would need a config reinterpretation.

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - producer-engine-choice-untested (class design, severity MEDIUM, open in rounds 1, 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- A term from the closed list in a stencil, spec or production Go literal outside the Claude engine's places fails the scan, the orch places are skipped, and an engine block with no copy for a registered engine fails (D2).
- Review: each Codex engine block and each per-engine Go text says what its Claude copy says, in Codex's terms (D2).

## PATTERN changes

- `PATTERN-stencil-ownership` — changes: Reading or writing a prompt, spec or stencil: read at call time via `stencilstore`; one engine's wording only in its engine blocks, rendered for the launch's engine; a spawned role's stencil renders the parent and edit directives. (test)
