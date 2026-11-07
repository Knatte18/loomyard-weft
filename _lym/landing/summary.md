# Take skills and effort for every role and subagent from agents.yaml

Which skills a role loads, and which effort a session or subagent runs at, were scattered Go lists and Claude Code defaults.
The gate fixer loaded only the language skills, so it edited Go files without `scribe:prose` or `scribe:code-quality`.
A subagent loaded a skill only if its brief named one.
A subagent whose Agent call named no `effort` ran at `xhigh`, whatever the operator's model settings said.
This change moves all three decisions into one hub config, `_lym/config/agents.yaml`.

## What changes

- New leaf package `internal/agentconfig` owns `agents.yaml`, with an embedded `template.yaml`.
  The `skills` key maps each role lym starts to its skill list, the gate fixer (`gate-fixer`) included.
  The Go lists `darnSkills`, `roleSkills`, `burlerSkills`, `fixerSkills` and `conflictSkills` are gone, and so is `languageconfig.AppendSkills`.
  Each engine now takes its list from the config by role, and `language.yaml`'s `skills` still follow it for a role that writes or reviews code.
- Load refuses a role without a list and a role key lym does not start.
  A role the hub file leaves out takes the template's list, `[]` loads none, and a role set to null is refused.
- The `efforts` key maps `default`, a model alias, a role, or `<role>/<model>` to an effort.
  The most specific key wins and `default` is required.
  A launch whose model-spec names no effort now gets an explicit `--effort` from this table, `OpenLaunch` included.
  A full model id such as `claude-haiku-5-5` resolves to its alias.
- Every Claude session's appended system prompt gets a subagent passage.
  It names the explicit `effort` each Agent call takes by model, and tells the session to name those of its own skills that fit the subagent's work in the brief.
  The passage is the one place the parent learns its list; the darn, plan, discussion, raddle and assist-fan stencils only point to it.
- The session's settings set `CLAUDE_CODE_SUBAGENT_EFFORT` to the role's effort on a model the table does not name.
  That is the backstop for a subagent whose call names no effort.
  Claude Code reads this variable as the default effort of a subagent no call or definition names one for.
- `shuttleengine.Config` gains an `Agents` field, loaded with `shuttle.yaml` from the same directories, so a failing `agents.yaml` fails the config load.
  Spec constructors in the burler, darn, webster, loom, seat, drift, treadle, friction, sampler, dobby, landingshed, mergeresolve, gatefixer and orch packages now name a config role instead of carrying their own list.

## Design decisions

- `skills` is classed Guard, since a skill list sets the rules a writer or reviewer works to.
  `efforts` is classed None, so a worktree's local override may tune it.
- The `efforts` map is open: a hub file's map replaces the template's whole, so a hub that sets it must keep `default`.
- The template sets `default: medium` and `fable: high`, so a session no longer inherits a launcher default it did not choose.
- A fork subagent shares its parent's effort, so the passage tells the session to give it none.

## To check by hand

- Not run: a live Claude Code session confirming that `CLAUDE_CODE_SUBAGENT_EFFORT` in the rendered settings makes a subagent with no `effort` run at the role's effort, and that the Agent call's explicit `effort` takes precedence over it.
  Check the `effort` field in the session transcript records.
- Not run: a gate-fixer run, to confirm it now loads `scribe:prose`, `scribe:code-quality` and `scribe:testing` ahead of the language skills.
- The template's per-role lists for roles that formerly had none (for example `describer`, `friction`, `drift-judge`) are new choices; review them in `internal/agentconfig/template.yaml`.
