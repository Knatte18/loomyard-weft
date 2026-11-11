# The orch session starts on opus at medium effort by default

The `orch` template held `model: ""` and `effort: ""`.
`orchCLI.orchSpec` passes those to shuttle as the session's `ModelChoice`, where empty means the provider default.
So the orch started on whatever Claude Code's default was on the machine: a local `/model`, `~/.claude/settings.json` or the account default.
Every other lym agent takes its model and effort from config.

The template now defaults to `model: opus` and `effort: medium`, so every hub and machine starts the orch the same way.
A hub that sets either key keeps its value.
An explicit empty value still means the provider default.

## Start paths

Every path that starts an orch session builds its spec through `orchSpec` from the orch config that `wireCLI` loads with `orchengine.LoadConfig`:

- `lym s` runs `lym adopt --here --no-orch`, then `lym beam start`, which calls `orchcli.BeamUp`;
- `lym beam start` calls `orchcli.BeamUp`;
- loom's beam-up step calls `orchcli.BeamUp`, and its dobby restart calls `orchcli.BeamUpWorktree`, which calls `BeamUp`.

Both the fresh launch and the resume in `launchOrch` use that spec, so none of these paths needed a change.

## Design decisions

- No `configreg.Reinterpretation` ships.
  The change gives an absent key a new default and leaves the meaning of every value a hub has set unchanged, including an explicit `""`.
- The test that loads the template with no config file now expects `opus` and `medium`.
