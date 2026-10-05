# Agents stop wrapping commands: env-file stdin, short step envelope

Agents wrapped commands for two reasons, and this change removes both.
A bare `cat > /tmp/x;` or `python3 -;` left reading the Bash tool's open stdin pipe hung sessions for up to 80 minutes.
The LLM driver wrapped every `lyx shed step` in `exec </dev/null; export LYX_TRACE_ID=…; … > /tmp/…; cut -c1-1000 …` to find its trace and shorten its output, and each background job's completion notice repeated that whole line into the driver's context.

## What changes

- **Stdin without a hook.**
  `claudeengine.Prepare` writes `bash-env.sh` (the single line `exec </dev/null`) into the run directory beside `settings.json`.
  The launch and resume pane lines export `CLAUDE_ENV_FILE` naming it, through `sh.WithEnv`, the way `CLAUDE_CODE_FORK_SUBAGENT` already rides the line.
  The PreToolUse(Bash) stdin rewrite hook (`bashStdinHookCommand`, `bashStdinPrefix`, `bashStdinRewriteJSON`) is deleted, so no per-call hook delays tool calls.
  Every lyx-spawned Claude session gets this, since all launch through `claudeengine.Prepare`.
  The webster fork-guard, Agent, AskUserQuestion and Stop hooks are unchanged.
- **Short step envelope.**
  `lyx shed step` writes the full envelope to its per-invocation record first, then prints one short JSON line carrying `envelope_path`, the record's path (the same path `lyx shed status` reports as `last_step.envelope_path`).
  The short success line keeps `run_id`, `producer`, `outcome`, `state`, `continue`, `reason`, `output`, `next`, `progress`, `history_length`, `trace_file` and `envelope_path`, plus `friction` and `parent_notice` when non-empty.
  The short error line keeps `error`, `kind`, `transient`, `run_id`, `trace_file` and `envelope_path`, plus `friction` when non-empty.
  The record holds the full envelope with today's key set.
  A new `--full` flag on the generic verb prints the full envelope on stdout instead; the `shedverbs` verb set carries it for every module with no per-module wiring.
- **Bare driver step.**
  `shed-template-driver.md` shows `lyx shed step {{.run_id}}` as the one command in a fenced block and says nothing else wraps it.
  The rule to run every `lyx` call as `(cd <drive-dir> && lyx …)` becomes a rule never to change the session's cwd.
  `friction_dir`, `scratch_dir` and `next_interrupt_policy` are now read from the file at `envelope_path`.

## Design decisions

- `exec </dev/null` in `CLAUDE_ENV_FILE` runs in the Bash tool's own shell ahead of each command, so it replaces only the default stdin.
  A heredoc, pipe or redirect attached to a command still supplies that command's stdin, and a child `bash` does not re-run the file.
  The file grants or denies nothing, so permission rules are untouched.
  An operator's own `CLAUDE_ENV_FILE` is overridden in lyx-spawned sessions only.
- The record is written before stdout is printed.
  When the step keeps no record (empty `Spec.StepsDir`, as in standalone `lyx shed`) or the record write fails, stdout carries the full envelope, so a short envelope never names a missing file.
  `--full` changes only what stdout prints; the record, exit code and run state are the same either way.
- `StepEnvelope` and `stepErrFields` still build the full envelope, and `StepKinds` and `continue`'s derivation are unchanged.
- The no-`cd` stencil rule is a prose guard.
  A bare step from another directory resolves the run from that directory's seed, so it refuses or steps a run seeded there under the same run-id; the stencil's report-refusals-verbatim rule covers the refusal case.
- `internal/shuttlecli/modelpin_test.go`: `wantSpawnSiteCount` drops by one, because the deleted stdin smoke test was a real-claude spawn site.
- Docs land with the code: the `claudeengine` doc and `settings.go` header, the `shedverbs` doc, `PATTERN-shed-verb-set`, `docs/overview.md` and the `loom step` help text.
  No new `PATTERN.md` invariant: the env file is a provider detail inside the existing shuttle provider seam.

## To check by hand

- **Live check, not run.**
  Build with `./deploy-dev`, launch a Claude session with the new binary, and confirm a Bash call reports `/proc/$$/fd/0` as `/dev/null` and the run's `settings.json` has no stdin hook.
  `CLAUDE_ENV_FILE` behaves as seen in Claude Code 2.1.289, and a release that drops or moves it brings the hang back silently, so this check is the only detection at landing.
- **Stencil deploy.**
  The board copy of `shed-template-driver` is behind the worktree source until a production deploy (`stencilstore` logs a `source-ahead` warning); the operator runs `update-plugins.sh`, nothing here does.
- **Integration tier.**
  `envfile_integration_test.go` replaces `bashstdin_integration_test.go`: it runs `bash -c` in Claude Code's prelude shape with an open stdin pipe, and shows a bare `cat` ends at once while a heredoc and a pipe into a child `bash -c 'cat'` still deliver their data.
  Run the `integration` tier for `internal/shuttleengine/claudeengine`.
- Drivers trained on the wrapper may keep it for a while despite the stencil; this change removes the reasons, not the habit.
