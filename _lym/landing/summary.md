# Skills and effort for every role and subagent come from agents.yaml

lym's config now decides which skills each role it starts loads and which reasoning effort its Claude sessions and their subagents run at.
Before this change, skill lists were Go variables spread over the spawning packages.
The gate fixer had no list, so it loaded only the language skills, without `scribe:prose` or `scribe:code-quality`.
A subagent got a skill only when its brief happened to name one, and a subagent spawned with no `effort` fell to Claude Code's default, seen as `xhigh` on Haiku workers.

## What changes

- **`agents.yaml`, a new hub-wide config module** owned by the leaf package `internal/agentconfig`:
  - `skills` maps every role lym starts to the skills it loads in its first turn, in order.
    `language.yaml`'s `skills` still follow it for a role that writes or reviews code.
    Load refuses a role set to null, a list that is not a list of plain strings, and a role key lym does not start.
    A role the hub file leaves out takes the template's list.
  - `efforts` maps `default`, a model alias, a role, or `<role>/<model>` to a Claude Code effort; the most specific key wins, and `default` is required.
    The template sets `default: medium` and `fable: high`, the same values `models.yaml` gives those models.
  - `skills` is classed Guard, so a local override can't set it; `efforts` is an open map classed None.
- **Shuttle resolves skills by role.**
  `shuttleengine.Spec` and `seatengine.Seat` gain `ConfigRole`.
  At Start, shuttle puts the role's list ahead of `Spec.Skills`, dropping repeats, and refuses a role without a list.
  `shuttleengine.LoadConfig` loads `agents.yaml` into the new `Config.Agents`.
  `gatefixer.SpawnOrAttach` sets the `gate-fixer` role itself.
- **Every spawn site names its role.**
  The Go lists are gone: darn, webster (master, recovery, plan fixer), gate fixer (the verify fixer included), merge fixer, conflict resolver, burler, PR reviewer, describer, fix review, plan, discussion and its advisors, raddle, the drift, treadle, bouncer and friction seats, orch, board tidy, driver, tackler, arbiter and sampler.
  Each site passes only the language skills, where it took them before.
  The orch's reload re-sends the list its wiring resolves from `agents.yaml`.
  `languageconfig.AppendSkills` is removed.
- **Effort for every Claude launch.**
  A launch or opened session whose model-spec names no effort gets `--effort` from the efforts table, by config role and model alias.
  `claude-haiku-5-5` matches `haiku`.
- **Subagents.**
  The Claude engine appends a passage to the system prompt of every session it launches.
  It names the explicit `effort` every Agent tool call takes, by model, and that a fork takes none.
  It also lists the session's own skills and tells the session to name, in each subagent's brief, those that fit the subagent's work.
  The darn, plan, discussion and raddle stencils and the Claude assist-fan block point at that passage instead of restating a list.
- **Backstop.**
  Claude Code's `CLAUDE_CODE_SUBAGENT_EFFORT` sets the effort of a subagent that no call or definition names one for.
  The settings document of every run lym launches sets it in `env` to the role's effort on an unlisted model.

## Design decisions

- **One field drives both lookups.**
  `ConfigRole` is separate from `Spec.Role`, since strand roles are dynamic (seat and bouncer roles carry prefixes) and `agents.yaml` needs a closed set to refuse a missing role.
  `agentconfig.Roles` is that closed set.
- **Effort is resolved in the Claude engine, not in shuttle.**
  The efforts table holds Claude Code vocabulary, so `claudeengine.Prepare` and `OpenLaunch` apply it.
  A Codex launch still runs at its model-spec's effort or Codex's default, since `codex.yaml` has its own effort vocabulary.
- **The backstop rides lym's per-session settings, not heddle's `.claude/settings.local.json`.**
  `CLAUDE_CODE_SUBAGENT_EFFORT` is an environment variable whose value here differs per role, while heddle's entries are static JSON.
  A session the operator starts by hand gets no backstop.
- **Behavior changes beyond the gate fixer.**
  The fix review, which loaded only the language skills, now loads `scribe:prose`, `scribe:code-quality` and `scribe:testing` first.
  The verify fixer moved from the webster list to the `gate-fixer` role, which holds the same three skills.
- MultiLLM recipe seats keep the skills their recipe rows name.
- Docs updated: `docs/overview.md`, `pattern/PATTERN-leaf-packages.md`, `pattern/PATTERN-config-layers.md`, and the package docs of `agentconfig`, `shuttleengine`, `claudeengine`, `languageconfig` and the migrated packages.

## To check by hand

- `CLAUDE_CODE_SUBAGENT_EFFORT` is documented only in Claude Code's changelog (2.1.316), with no stated precedence against an agent definition's `effort`.
  In a launched session, check that a subagent spawned with no `effort` runs at the role's effort.
- Check that a session's model follows the appended passage: every Agent call carries `effort`, and a code subagent's brief names the skills.
