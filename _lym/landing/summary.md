# Start the status-bar agent with its context and state the name rule

The `agent` status-bar button had two problems.
It refused a name with spaces without saying what a valid name is,
and the session it opened knew nothing of its hub, worktree, purpose or skills.

## What changes

**Names.**
The `agent` and `board` buttons ask for lowercase letters, digits and dashes, and say that spaces become dashes.
The question leaves out that a name must start with a letter; the refusal message states it.
`lym shuttle open --role` and the new `lym shuttle agent --name` lower a typed name and join its words with dashes,
so `My Agent` becomes `my-agent`.
A name still outside the role grammar is refused with a message that states the rule.
The button does not ask again after a refusal.

**A new verb, `lym shuttle agent`.**
The `agent` button now runs `lym shuttle agent --name {answer} --model 'opus[high]'` instead of `lym shuttle open`.
It starts an interactive session whose opening prompt says what the agent is, which hub and worktree it sits in,
and which board entry the worktree serves or that it is the prime.
It also says what the agent is for: explaining lym, running its verbs, reading the code,
and changing or committing it on the operator's word.
Its limits keep the agent in that worktree unless the operator names another directory, and forbid pushing.
The session gets the settings, shell guards, standing passages and skills of a run session:
the `agent` role's list in `agents.yaml` (`scribe:prose`, `scribe:conversation`, `ly:cli`, `ly:board`),
then the hub's language skills.
The `board` button still uses `lym shuttle open`.

The prompt is the new `operatoragent-template-start` stencil, filled by the new `internal/operatoragent` package.

## Design decisions

- **A run session, not an opened one.**
  `lym shuttle open` sends no prompt, so the verb goes through `Runner.StartSession` with an unawaited spec.
- **Skills come from the launch spec.**
  The stencil does not ask the agent to load any.
- **`Spec.EndsWithSession`.**
  Nobody waits on this session to stop it, so its pane line touches an exit marker, removes its own strand and exits the shell.
  The marker is the strand's done-when list, so a resume drops a closed agent.
  The run record reads running until the orphan sweep removes the run directory.
  `pattern/PATTERN-shuttle-provider-seam.md` records this exception for strand removal and the done-when list.
- **`Spec.NoCarryOver`.**
  The start command exits at once, so an earlier agent that is still live would read as an unfinished predecessor,
  and a second agent of the same name would inherit what the operator typed into the first.
  The spec has an empty role key, so it neither takes operator instructions over nor leaves them to a later run.
- **Refused under a pause.**
  A start normally waits a pause out, which a button click would show as nothing happening.
  The check `Runner.Open` already made is now `Runner.RefuseCovered`, shared by both.
- **The operator's rules override the parent section.**
  The prompt keeps the standard parent directive, followed by rules that send questions to the operator in the pane,
  ask for no report, and keep only the section's creel mail rules.

Two changes reach beyond the new verb:

- A strand reed no longer knows now counts as removed in the startup teardown of every run.
- A resume line that only exits the shell is left out of the pane line, for opened sessions too,
  so the strand is still removed.

## To check by hand

The button and the new verb were not tried in a live session.

- Click `agent` in the prime and in a task worktree.
  Check the prompt, the loaded skills, and that the window closes when the session exits and stays closed after a reed resume.
- `paneRules` in `internal/operatoragent/spec.go` overrides the parent directive that
  `contracts/stencils/operatoragent/operatoragent-template-start.md` renders.
  Check that the rendered prompt reads consistently both with and without a recorded parent.
- `EndsWithSession` ends the agent's session (`internal/shuttleengine`).
  Its tests inspect the pane lines against a fake reed without running them, so check the end of a real session.
  A provider that dies during startup takes its pane with it,
  so the saved capture holds only what an earlier startup probe took.
- `TestAgentCmd_StartsSessionFromGeometry` covers the spec `lym shuttle agent` builds
  and its `--name` and `--model` refusals (`internal/shuttlecli/agent.go`).
  It stops at the engine's prepare step, and no test drives the verb under a pause.
- Rows in `internal/shuttlecli/cli_test.go` cover a normalized and a refused `lym shuttle open --role`
  (`internal/shuttlecli/open.go`).
  Check that they are enough.

## Merge danger

**Door:** two-way
**Blast radius:** shuttle
