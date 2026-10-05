# Support log — agent-command-wrapping

## Interview

Autonomous mode: no operator answered, so each turn below is a question the agent put to itself and settled from the code, the board and the installed Claude Code binary.

1. Is the task real and bounded?
   The board entry (#128) merges `agent-stdin-hang` and `driver-step-clutter`; its done-when names three outcomes: no stdin hook (or a JSON-parsing one), a short step envelope, a stencil showing the bare call.
   Related entries read: `agent-permission-setup` (wants a python deny hook, out of scope), `test-suite-prune-core` and `test-suite-prune-rest` (own the test packages touched here).
2. Why does a bare `cat` hang at all?
   Read in the Claude Code 2.1.289 bundle: the Bash tool spawns its shell with `stdin: "pipe"` and appends `< /dev/null` only when the command has no heredoc and no `<` redirect.
   So the hang needs a heredoc or redirect elsewhere in the same command, which matches the transcripts (`cat > /tmp/x; python3 - <<EOF`).
3. Is there a hook-free place to cut stdin?
   The same bundle reads `CLAUDE_ENV_FILE` once per session and pushes its content into the per-command prelude before `eval`, in the same shell.
   `exec </dev/null` there cuts the default stdin and is not inherited as a startup file by child shells, unlike `BASH_ENV`.
   This session's own process confirmed the prelude shape and that stdin is a pipe absent the hook prefix.
4. Where to set it?
   `buildLaunchCmd`/`buildResumeCmd` already set `CLAUDE_CODE_FORK_SUBAGENT` through `sh.WithEnv` because reed scrubs the server env; the same route covers launch and resume.
   orch, burler, loom, webster and shuttle all launch through `claudeengine.Prepare`, so one site covers every lyx-spawned session.
5. What should the short step envelope keep?
   Every field the driver stencil branches on per step, plus `trace_file` (read after every step) and a pointer to the full record.
   `friction_dir`, `scratch_dir` are used only at parking and repair; `next_interrupt_policy` is read from status; `status_file` and `trace_id` are not used by the driver.
6. Can the short envelope point at a record that might not exist?
   The recorder logs and ignores write failures and keeps nothing without `StepsDir`; decided to write the record first and fall back to the full envelope on stdout when no record exists.
7. Does the stencil's subshell rule conflict with "the one exact command"?
   Yes: `(cd <drive-dir> && lyx …)` is itself a wrapper; replaced by a no-`cd` rule since the session's cwd is the drive directory.
8. Testing: unit tests for settings and launch lines, an integration test reproducing Claude Code's prelude shape, shedverbs tests for the short, full and fallback envelopes, and one live check recorded in the PR, since no test may run `claude -p`.

## Rejected alternatives

- **Keep the hook and parse JSON** (the board's fallback): a per-call hook still delays every tool call, which the operator dislikes; only needed if no hook-free way worked, and one does.
- **`BASH_ENV`**: sourced by every non-interactive child `bash`, so `… | bash script.sh` would lose its piped stdin unless the file also unset `BASH_ENV`; also unclear for `bash -c -l` login shells. `CLAUDE_ENV_FILE` has neither problem.
- **Changing how shuttle launches Claude Code** (stdin of the `claude` process): the Bash tool's shell gets a fresh pipe from Claude Code regardless of Claude's own stdin, so this cannot help.
- **Setting `CLAUDE_ENV_FILE` through settings.json's `env` key**: plausible, but the launch-line route is already proven against reed's env scrubbing and needs no assumption about when settings env reaches `process.env`.
- **A timeout on Bash calls**: ends the hang after the fact; the board rules it out.
- **A human-text step output instead of JSON**: breaks the JSON-envelope contract (`PATTERN-cli-cobra`) and the uniform success/error shape the driver parses.
- **A new `next_action` field**: `continue` already is the next action; YAGNI.
- **Dropping `trace_file` from the short envelope**: the driver reads the trace after every step, and its absence was the likely motive for minting `LYX_TRACE_ID`.
- **Setting `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR` so cwd cannot drift**: would change every agent's cd behavior, beyond this task.
- **A python deny hook** (`agent-permission-setup`): separate note, and adds a hook while this task removes one.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): hook-free stdin mechanism → `CLAUDE_ENV_FILE` with `exec </dev/null`, set on launch and resume lines. Rationale: read from the Claude Code 2.1.289 bundle; same shell as the command, not inherited by child shells.
- Q2 (resolved, auto-pick): keep the hook as a fallback? → no; delete it entirely. Rationale: YAGNI; the live check at landing detects a non-working env file.
- Q3 (resolved, auto-pick): env file location → run directory beside `settings.json`, named `bash-env.sh`. Rationale: ephemeral, per run, survives resume.
- Q4 (resolved, auto-pick): short envelope key set → D2 lists. Rationale: everything the driver branches on per step plus `trace_file` and `envelope_path`.
- Q5 (resolved, auto-pick): full envelope access → its per-step record plus a `--full` flag. Rationale: the record already exists and status already points at it; the flag serves humans and tests.
- Q6 (resolved, auto-pick): record write ordering → full record written before stdout; full envelope on stdout when no record is kept. Rationale: a short envelope never names a missing file.
- Q7 (resolved, auto-pick): driver cwd rule → no `cd` in the driver session, `lyx` calls bare. Rationale: the subshell form is itself a wrapper.
- Q8 (resolved, auto-pick): PATTERN entry → none. Rationale: provider detail inside an existing seam.
- Q9 (resolved, auto-pick): live verification → one recorded check from a `./deploy-dev` binary in the PR. Rationale: no test may spawn `claude -p`, and the mechanism depends on Claude Code internals.
