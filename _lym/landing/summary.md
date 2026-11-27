# Agent button: state the name rule and start the agent with hub context

The agent button in the reed status bar had two faults.

1. It asked for `Agent name:` and passed the answer to `lym shuttle open --role` unchanged.
   A name with a space failed reed's role check, and the client showed the head of a JSON error line,
   so the rule, which came at the end of that line, never showed.
2. `lym shuttle open` launched a bare session: no prompt, no shell guard, no standing passages.
   The agent knew nothing of lym, its hub, its worktree or what it was for.

## Changes

- `internal/agentname`: `RoleFromText` lowers a typed role and joins its words with dashes, so `My Agent` becomes `my-agent`.
  `RoleRule` states the role grammar in words.
- `internal/shuttlecli`: `lym shuttle open --role` runs its value through `RoleFromText`.
  A value still outside the grammar is refused with the rule at the front of the message:
  `--role "my_agent" is no valid agent name: use lowercase letters, digits and dashes, starting with a letter, as in my-agent`.
- `internal/shuttlecli/agent.go`: a new verb, `lym shuttle agent --name <name> [--model <spec>]`, defaulting to `agent`.
  It normalizes the name the same way and builds its spec through `internal/operatoragent`.
  It starts the session through `Runner.StartSession`, the unawaited interactive path the PR-Review and sampler sessions use,
  so the session gets the settings, shell guards, standing passages and skills every run session of its engine gets.
- `internal/operatoragent`: a new package that builds the spec and renders the opening prompt.
  The prompt, the new `operatoragent-template-start` stencil, tells the agent it was started from the agent button,
  what lym is, its hub's shortname and directory, its worktree,
  and whether the worktree serves a board entry, which it names, or is the prime.
  It renders the parent and edit directives, and the implementer PATTERN directive when the worktree has a PATTERN overview.
- `internal/agentconfig`: a new `agent` role, whose shipped skill list is `scribe:prose`, `scribe:conversation`, `ly:cli` and `ly:board`.
  The session loads it, followed by the hub's language skills.
- `internal/reedengine` templates: the agent buttons of the task, beam and prime groups run `lym shuttle agent --name {answer}`.
  Their ask text, and the board button's, now names the rule:
  `Agent name (lowercase letters/digits/dashes; spaces become dashes):`.
  An ask text may hold no comma, which is why the rule uses slashes.
- `internal/shuttleparity`: the new spec producer has a row, so its Codex manifest is held to Claude's.
- `docs/overview.md`: a module row for `internal/operatoragent` and the new verb in the shuttle entry.

## Design decisions

- The fix lowers the name and turns its spaces into dashes; it does not raise the prompt again after a refusal.
  A name that still fails, such as `my_agent`, is refused with the rule shown first.
- The agent runs as an unawaited run session, not an opened one.
  `Runner.Open` gives a session creel delivery alone.
  `StartSession` is the existing path that gives a session everything a run session gets, on either engine.
- The skills reach the session through the spec's config role and language skills, never through prompt text,
  since no stencil may tell an agent to load a skill.
- The parent directive uses the worktree's recorded parent.
  In the prime that is empty, so the no-parent variant renders, as it does for any session with no recorded parent.
- The board button keeps `lym shuttle open` with its own skill list.
  Only its ask text changes, and it gets the same name normalization through `--role`.
- A hub whose layered `reed.buttons` groups replace the shipped defaults keeps its own button texts.
  Such a hub changes its `run` lines to `lym shuttle agent` itself to get the new session.

## Tests

- `TestRoleFromText` (`internal/agentname`): spaces, upper case and repeated space normalize to a valid role,
  and a refused character or a leading digit stays refused.
- `TestSessionSpec` (`internal/operatoragent`): the task worktree's and the prime's spec are interactive, unawaited and focused in a window of their own,
  carry the `agent` config role, the chosen name, the model choice and the language skills,
  and their prompt names the hub, the worktree and the board entry or the prime.
  Each required input is refused by name when empty.
- `TestLoadConfig_TemplateDefaultsResolve` (`internal/reedengine`) pins the new button rows.
- The parity row for `internal/operatoragent#SessionSpec` prepares the spec on both engines.

## Merge danger

**Door:** two-way
**Blast radius:** reed
