# codexengine: shell guard rows as Codex hooks, and skills loaded as files

Codex sessions now get the same shell guards Claude sessions get, and they can load skills.
This covers decisions D9 and D10 of the `codex-engine` project's Sketch.

## What changes

- **Shell guards on Codex (D10).** Each installed row of `internal/shellguard`'s table is added to the Codex launch line as a `PreToolUse` hook matching `Bash`.
  The rows' order is the table's.
  A matching command prints Codex's deny decision with the row's steer, the shape the spike tested.
  Any other command prints the empty hook output.
  All entries share one `-c hooks.PreToolUse=[…]` setting, because a second `-c` with the same key would replace the first.
  The python row follows the shuttle config's python switch, the publish row follows the spec's `DenyPublish`, and the refuse-raw row takes its list from the hub's gate config, as on Claude.
  A gate config that exists but fails to load refuses the launch.
- **Deny notice.** The run's prompt file now opens with the notices of the installed rows.
  The standing passages and the prompt come after them.
- **Per-engine row texts.** `shellguard.Row` gains `EngineTexts`, keyed by engine registry name, along with `SteerFor` and `NoticeFor`.
  These return the engine's own text when the row has one and the plain text otherwise.
  The python, shell-edit and headless-claude rows carry Codex texts.
  These name Codex's `apply_patch` tool where the plain texts name Read, Edit and Write, and Codex's own subagents where they name the Agent tool.
  `checkTexts` now checks engine texts too, so `shellguard`'s init refuses a forbidden character in them.
  Claude's rendering keeps the plain fields and is unchanged.
- **Publish guard.** Prepare no longer refuses a spec with `DenyPublish`, since the publish row is now installed on Codex.
- **Skill loading (D9).** `codexengine` implements `SkillLoader`.
  A skill `<plugin>:<name>` resolves under `codex.yaml`'s `skill_roots`, which by default is the Claude plugin cache.
  It resolves to the newest version's `skills/<name>/SKILL.md`, with version folders compared field by field and numbers compared as numbers.
  The load line asks the model to read each resolved file by absolute path in one turn.
  A skill that no root holds is left out of the line and reported unknown.
  The load turn is classified from the rollout that the turn end's `transcript_path` names.
  The turn starts at the latest user message that equals the load line, which can be a `response_item` message of role user or a `user_message` event.
  A skill counts as loaded when a later tool call (`function_call`, `custom_tool_call` or `local_shell_call`) names its `skills/<name>/SKILL.md` tail, and as missing otherwise.
  A rollout that cannot be read, or that holds no matching user message, gives an unverified report.
  A spec that names skills launches with the load line as its first turn and returns the prompt pointer as `Launch.PromptLine`.

## Design decisions

- The Codex key in `shellguard` is its own unexported constant, `engineCodex`.
  `shellguard` cannot import `codexengine`, which also holds `engineName`.
  The codexengine tests read the rendered commands and the notice through `engineName`, so a mismatch between the two shows up as a failure.
- Every guard command reads the payload once and runs one `grep -E` per pattern, whether the row has one pattern or several.
  Each command exits 0, so a non-match never blocks the call.
- The foreground phrase in the gate rule for Codex is "with the shell command timeout set to 3600000 milliseconds".
  It names no Codex tool parameter, since the spike recorded none.
- Classification uses the rollout's user message, not the Stop payload's `turn_id`.
  The spike recorded no turn id on rollout tool calls, but it did record the user message.

## To check by hand

- In one real Codex session, confirm that several `PreToolUse` entries with matcher `Bash` all run, and that any one deny blocks the command.
  The spike tested a single entry.
- Confirm that Codex 0.164's file edit tool is `apply_patch`, as the Codex texts state.
- Load a skill in a real Codex session and confirm the rollout records the read as a `function_call` whose arguments hold the `SKILL.md` path.
  The tests use rollout lines shaped like the bench harness's `exec` calls.
