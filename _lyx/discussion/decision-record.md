# Decision record — agent-command-wrapping

## Goal

Agents stop wrapping their commands, because lyx removes both reasons they wrap.
A Bash call in any lyx-spawned Claude session reads `/dev/null` as its default stdin with no per-call hook.
`lyx shed step <run-id>` prints an envelope short enough to read whole, and the driver stencil shows that bare call as the one way to step.

## Scope

In:

- `internal/shuttleengine/claudeengine`: drop the PreToolUse(Bash) stdin rewrite hook (`bashStdinHookCommand`, `bashStdinPrefix`, `bashStdinRewriteJSON`) and replace it with a `CLAUDE_ENV_FILE` that Prepare writes into the run directory and the launch and resume lines export.
- `internal/shedverbs`: `step` prints a short envelope by default, keeps the full envelope in its per-invocation record, and gains a `--full` flag.
- `contracts/stencils/shed/shed-template-driver.md`: shows the exact bare step command and says where the trace and the full envelope are.
- Package docs (`claudeengine` doc and `settings.go` header, `shedverbs` doc) and the tests that cover the changed behavior.

Out:

- Denying `python`/`python3` through a hook (`agent-permission-setup` note): a separate note, and contrary to this task's direction of removing hooks.
- Any timeout that ends a hang after the fact.
- The webster fork-guard Bash hook, the Agent and AskUserQuestion hooks, and the Stop hook: unchanged.
- Changing which fields `lyx shed status` reports.
- Pruning or refactoring tests beyond those covering the changed behavior (owned by `test-suite-prune-core` and `test-suite-prune-rest`).

## Decisions

### D1 — Stdin comes from `CLAUDE_ENV_FILE`, not a hook

Decision: Prepare writes a file `bash-env.sh` into the run directory containing `exec </dev/null`, and `buildLaunchCmd` and `buildResumeCmd` set `CLAUDE_ENV_FILE` to its absolute path through `sh.WithEnv`, the way `CLAUDE_CODE_FORK_SUBAGENT` already rides the pane command.
The PreToolUse(Bash) stdin rewrite hook and its constants are deleted; every lyx-spawned Claude session (loom producers, webster, burler, orch, shuttle) gets the change, since all launch through `claudeengine.Prepare`.

Rationale: the Bash tool leaves the shell's default stdin an open pipe, and appends `< /dev/null` only to a command with no heredoc and no `<` redirect, so a bare `cat > /tmp/x;` sharing a command with a heredoc reads that pipe and hangs: the observed shape.
The content of `CLAUDE_ENV_FILE` runs in the same shell, ahead of the agent's command, so `exec </dev/null` there replaces the default stdin before the command runs.
A heredoc, pipe or redirect attached to a command still supplies that command's stdin, and a child `bash` the agent starts (`… | bash script.sh`) does not re-run the file and keeps its piped stdin.
It costs nothing per call, and the run directory already holds the run's `settings.json`, so a resume finds the file where the launch left it.
The variable is the provider's own mechanism, so it stays inside `claudeengine` (`PATTERN-shuttle-provider-seam`), and the command string is built only through `internal/shell` (`PATTERN-shell-mechanics-seam`).

Bound: the file runs one `exec </dev/null` and nothing else; it changes only the default stdin of the tool's own shell, never a stdin a command attaches, and grants or denies nothing, so permission rules are untouched.
A `CLAUDE_ENV_FILE` the operator's environment already exports is overridden for lyx-spawned sessions only.

### D2 — `lyx shed step` prints a short envelope; the record keeps the full one

Decision: by default `step` prints one JSON line through `internal/output` with these keys:

- success: `run_id`, `producer`, `outcome`, `state`, `continue`, `reason`, `output`, `next`, `progress`, `history_length`, `trace_file`, `envelope_path`, plus `friction` and `parent_notice` when non-empty;
- error: `error`, `kind`, `transient`, `run_id`, `trace_file`, `envelope_path`, plus `friction` when non-empty.

`envelope_path` is the step's own record `<steps_dir>/<trace_id>.json`, the same path `lyx shed status` reports as `last_step.envelope_path`, and it holds the full envelope that `StepEnvelope` and `stepErrFields` build today, unchanged in key set.
The full envelope is written to the record before stdout is printed; when the step keeps no record (empty `Spec.StepsDir`, as in standalone `lyx shed`) or the record write fails, stdout carries the full envelope instead, so a short envelope never names a missing file.
A new `--full` flag on the generic `step` verb prints the full envelope on stdout.
The closed `StepKinds` vocabulary and `continue`'s derivation are unchanged.

Rationale: the clutter is the long absolute paths (`status_file`, `friction_dir`, `scratch_dir`) and fields the driver reads elsewhere (`next_interrupt_policy` is read from status as `interrupt_policy`; `trace_id` from `last_step`).
The short line keeps every field the driver branches on at each step, plus `trace_file`, which it reads after every step, so a bare call's completion notice is readable without `cut` or a temp file.
The fields needed only at a stop or a repair stay one read away in the record the step already keeps.
The flag lives in `internal/shedverbs`, so every module arming the verb set gets it with no per-module wiring (`PATTERN-shed-verb-set`).

Bound: `--full` changes only what stdout prints; the record, the exit code and the run's state are the same with or without it.

### D3 — The driver stencil shows the one bare call

Decision: `## How to invoke a step` in `shed-template-driver.md` shows a fenced block holding exactly `lyx shed step {{.run_id}}`, run as one background job from the session's cwd, and states that this is the whole command: no `exec`, no `LYX_TRACE_ID`, no redirect, no temp file, no `cut`, no script file.
It says the printed envelope is the whole output to read, `trace_file` names the trace and `envelope_path` the full envelope.
The `## Drive directory` rule changes from wrapping every `lyx` call in `(cd <drive-dir> && lyx …)` to never changing the session's own cwd: `lyx` calls run bare from it, and a command that needs another directory uses `git -C` or its own `(cd … && …)` subshell.
`## Error envelopes`, `## Parking` and `## Repair records and the stop report` say that `friction_dir`, `scratch_dir` and `next_interrupt_policy` are read from the file at `envelope_path`, or from the printed envelope itself when it carries no `envelope_path` (the D2 fallback).
The `When the job left no output` fallback through `last_step.envelope_path` stays, and so does the sentence that a wrong directory surfaces as the recipe's own refusal text, reported verbatim.

Bound: the no-`cd` rule is a prose guard, not a structural one.
A bare `lyx shed step <run-id>` from another directory resolves the run from that directory's own seed, so it refuses (missing seed, ownership) or steps a run seeded there under the same run-id; the run-id the stencil names bounds it, and the stencil's refusal-verbatim rule reports the refusal case.

Rationale: the wrapper outlived the instruction that once prescribed it, and the likely motive (minting an id to find the trace) is answered when the envelope itself names the trace.
A literal block is the shape a model copies; the subshell rule was itself a wrapper around every call, and the session's cwd already is the drive directory.
The stencil stays recipe-blind and keeps its markers.

### D4 — Docs land with the code

Decision: the `claudeengine` doc and `settings.go` header describe the env file instead of the rewrite hook; the `shedverbs` doc (and `StepEnvelope`'s comment) describes the short envelope, the record-first write and `--full`.
No `PATTERN.md` entry is added: the env file is a provider detail inside the existing seam, not a cross-cutting invariant.
`docs/overview.md` is unchanged, since no module or execution-stack layer changes.

Rationale: CLAUDE.md requires observable CLI changes and their module docs in one commit.

## Constraints

- `PATTERN-shuttle-provider-seam`: `CLAUDE_ENV_FILE` and the file's content live only in `internal/shuttleengine/claudeengine`.
- `PATTERN-shell-mechanics-seam`: the launch and resume lines gain the variable only through `shell.Shell.WithEnv`.
- `PATTERN-durable-vs-ephemeral-state`: the env file is ephemeral and sits in the run directory beside `settings.json`, which is already under `.lyx`; no new `_lyx`/`.lyx` literal is spelled (`PATTERN-lyxdirs-single-declarer`).
- `PATTERN-shed-verb-set`: the short envelope and `--full` are in `internal/shedverbs`, which derives no path; `envelope_path` is built from the told `Spec.StepsDir`.
- `PATTERN-cli-cobra`: errors stay JSON envelopes through `internal/output`.
- `PATTERN-parent-directive` and `PATTERN-producer-pointer-rule`: the driver stencil keeps its parent-directive marker, asks no operator, and points at `contracts/specs/refusal-spec.md` rather than restating it.
- `PATTERN-test-tier-purity`: a test that spawns `bash` is `integration`-tagged; `PATTERN-test-parallel` and `PATTERN-test-economy` apply to every new or changed test.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule for `.md` files.
- Coordination: `internal/shuttleengine` tests belong to `test-suite-prune-core` and `internal/shedcli` tests to `test-suite-prune-rest`; touch only the tests that cover the changed behavior.
- Windows: Claude Code runs the Bash tool under git-bash there, where `exec </dev/null` is valid; the pwsh `WithEnv` form is session-wide, which is acceptable for a per-run pane.

## Auto-mode assumptions

- No operator answered; every choice below was self-picked (see the support log's question ledger).
- `CLAUDE_ENV_FILE` keeps the behavior observed in Claude Code 2.1.289: read from the process environment and its content prepended to every Bash command's shell, unless the session scrubs credential env (sandbox mode, which lyx agents do not use).
  A release that changes this brings the hang back, as the first Open risk states.
- No operator workflow depends on its own `CLAUDE_ENV_FILE` inside lyx-spawned sessions.
- No Go code parses `lyx shed step`'s stdout for `status_file`, `friction_dir`, `scratch_dir`, `trace_id` or `next_interrupt_policy`; tests that do are updated to read the record or pass `--full`.

## Open risks

- A future Claude Code release may drop or move `CLAUDE_ENV_FILE`; the hang then returns silently.
  The live check in the acceptance criteria detects it at the time of landing only.
- Drivers already trained on the wrapper may keep it for a while despite the stencil; the task removes the reasons, not the habit itself.
- A driver whose cwd drifts after a `cd` runs bare `lyx` calls from the wrong directory, bounded as D3 states; the stencil's no-`cd` rule is the only guard.

## Acceptance criteria

- No settings document `claudeengine` builds carries the Bash stdin rewrite hook, in fork and non-fork, interactive and autonomous runs; the webster fork guard is still present in fork runs.
- Prepare writes the env file into the run directory with `exec </dev/null` as its content, and both the launch and the resume command lines set `CLAUDE_ENV_FILE` to its absolute path, for the POSIX and pwsh shells.
- An integration test runs `bash -c` in Claude Code's prelude shape (env-file content, then `eval` of the command) with a stdin pipe held open, and shows: a bare `cat` ends at once with empty input; a heredoc attached to a command is still read; a pipe into a child `bash -c 'cat'` still delivers its data.
- A live check, recorded in the PR: a Claude session launched by the new binary (built with `./deploy-dev`) reports `/proc/$$/fd/0` as `/dev/null` in a Bash call, and its `settings.json` has no stdin hook.
- `lyx shed step` with a record kept prints the short success envelope with exactly the D2 keys, and the file at `envelope_path` holds the full envelope with today's key set.
- An error step prints the short error envelope with the D2 keys, and its record holds the full error envelope.
- `--full` prints the full envelope on stdout; the record is identical with and without it.
- A step with no record kept (empty `StepsDir`) or whose record write fails prints the full envelope.
- `lyx shed status`'s `last_step.envelope_path` names the same file the step's short envelope named.
- The driver stencil shows `lyx shed step {{.run_id}}` as the one command in a fenced block, contains no `(cd <drive-dir> && lyx` form, states the never-change-cwd rule, says where to read `friction_dir`, `scratch_dir` and `next_interrupt_policy` when the envelope carries no `envelope_path`, and names `trace_file` and `envelope_path`; the existing driver-stencil tests (`cmd/lyx/driverstencil_test.go`, `internal/loomcli/driverprompt_test.go`) pass or are updated for the changed text.
- `go test ./...` and the `integration` tier pass for the touched packages.

## Notes for the plan writer

- The record writer today tees stdout (`stepRecorder.tee`/`finish` in `internal/shedverbs/steprecord.go`); D2 needs it to write the full envelope independently of what stdout prints, before printing.
- `bashstdin_integration_test.go` pins the deleted hook and is replaced by the D1 integration test.
- `internal/loomcli/step_test.go` calls `StepEnvelope` directly for `next_interrupt_policy`; that function keeps building the full envelope.
- Claude Code's prelude shape seen in a live session: `/bin/bash -c source <snapshot> 2>/dev/null || true && shopt -u extglob … && eval '<cmd>' && pwd -P >| /tmp/claude-<id>-cwd`, with the env-file content in the prelude before the eval; the integration test reproduces this shape.
