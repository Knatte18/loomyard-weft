# Session scope: drop the Workflow tool and dead plugin skill overrides

A spawned role's first request falls from 37,330 tokens to 29,086 with the scope on, and from 49,611 to 41,367 after the skill preload.
Both figures come from a real `lym shuttle open` in the sandbox hub `lyt` on Claude Code 2.1.319, with the launch trial report's role, model and preloads, measured with the same `measure.sh`.
Tool schemas fall from 86,589 to 64,249 characters.
The orch was not relaunched: it gets `disableWorkflows` the same way, and a unit test row pins that.

## The larger Workflow schema

The cause is `disableBundledSkills`, not `disableClaudeAiConnectors`, the report's lead.
In the binary, the `Workflow` tool points to the bundled `workflow-authoring` skill only while bundled skills are on;
with them off it inlines the whole authoring reference in its schema.

The scope now also sets `disableWorkflows`, which removes the tool, its base schema included.
The launches show `Workflow` absent from the tool list.
No session lym launches uses it: no stencil, spec or Go file names it, and none of the 4,136 Claude Code transcripts of the last seven days calls it.
So every session loses it, and no role is exempt.

## Plugin skills stay listed

No setting hides a single skill of a plugin that is on, so this fix takes the task's fallback.
What was tried:

- `skillOverrides`: the binary reads it for a plugin's skill only through its claude.ai skill sync, which an installed plugin such as `scribe` does not follow.
- `permissions.deny` with `Skill(<name>)`: a launch with a deny rule for all twelve unneeded `scribe` skills still listed all fourteen. The rule refuses a call and leaves the listing.
- The session skills allowlist: the binary sets it only from an SDK session's initialize message, which an interactive session cannot send.

The `skillOverrides` entries derived from each plugin are gone, and with them the reading of each plugin's skill directories and the `ly` exception.
`readInstalledPlugins` now returns plugin ids alone.

What stays listed: every skill of a plugin that is on, so a session that preloads one `scribe` skill lists all fourteen, about 530 tokens.
A plugin that is off stays out whole, through `enabledPlugins`.
The bundled-skill listing also keeps `ladder-run` and `plugin-authoring`, about 120 tokens.

## Design decisions

`skills_off` and its `anthropic-skills:*` entries stay.
The first launch left them out and the listing came back at 3,236 tokens, so these overrides do the hiding, not `disableClaudeAiConnectors`.
This settles the row the report left "not isolated".
The block's comment now says a plugin's skill named there stays listed.

No config reinterpretation ships: `skills_off` keeps its meaning, and no hub tunes `session_scope`.

A per-session plugin directory through `--plugin-dir`, holding only the preloaded skills, might hide the rest.
It is a launch mechanism rather than a setting, and it would need to carry each plugin's hooks and files, so it is left out.

## Check

Two of the four allowed launches were used, both interactive and with a throwaway build of this branch.
The first tried `disableWorkflows` with the deny rules and without the `anthropic-skills` overrides;
the second ran the committed code.
In both, `scribe:prose` and `scribe:code-quality` loaded through the Skill tool and their bodies are in the second request.
The trial sessions are closed.
`go vet` passes on the package; the tests run at the verify gate.
