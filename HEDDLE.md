# Heddle

Every entry heddle delivers, grouped by the target it reaches, each group in render order.
lym rewrites this file whole at every heddle write; change an entry with `lym heddle set`, never this file.

## `claude-file`

Rendered into `CLAUDE.local.md` at every worktree's anchor.

### `lym-batching`

- From: shipped with lym
- Order: 10
- Targets: `claude-file`

```markdown
## Batch your calls

- Request every call whose arguments you already know in one message.
- Before you look anything up, list every question the step raises.
- Send every grep and read for all of those questions in one message.
- Wait for a result only to choose the next call.
- When a result arrives, send every call it unlocks and every call you already know in one message.
- Read what you need before you edit: a file whole, or every span you need from it, in one message.
- Read a file whole, or every span you need from it, even though the Read tool advises reading only the part you need.
- Never grep for a line number and then read a slice around it.
- Never read one file in slices over several turns.
- Keep a file you have read in mind; read it again only when something other than your own edit has changed it.
- After a command rewrites files, such as a formatter, a generator or a script, read every file it changed again in one message before you edit any of them.
```

### `lym-plan-edits-before-gate`

- From: shipped with lym
- Order: 20
- Targets: `claude-file`

```markdown
## Make every round of edits in this order

- Follow the steps below in order every round, and never deviate from them.
- Count as a round any set of edits, whatever starts it: a card, a task, a plan, a review's findings, a fixer's report or a failed gate.

1. Read everything the round needs in one message: every Read and grep call together, a file whole when it fits and every span of it you need otherwise.
   - Make a second round of reads only for files the first round named.
2. Decide every edit across every file the round touches before you make any of them.
3. Make all of those edits in one message, every Edit and Write call together in one turn.
4. Run the round's gate once, or its validator when it has no gate, and fix everything its report shows the same way:
   1. Read what the fixes need in one message.
   2. Decide every fix.
   3. Make all of them in one message.
   4. Run the gate or validator again.

- Never read one file per turn.
- Never spread an edit, a read and another edit over turns.
- Never run the gate between the edits of one round.
```

### `nicknames`

- From: the hub's heddle.json
- Order: 900
- Targets: `claude-file`

```markdown
## Nicknames

- Read "Sketch" (capital S) as the decision record: the approved record a Discussion writes, stored on the board for a discussed entry.
- Read "Merriam" as webster's long-lived master session, which forks one implementer per batch.
- Use every nickname in this list freely with the operator.
- Keep the formal names in code, files, verbs and specs.
```

## `codex-file`

Rendered into `AGENTS.override.md` at every worktree's anchor. It also carries every hub entry aimed at `claude-file`.

No entry.

## `claude-settings`

Merged into `.claude/settings.local.json` at every worktree's anchor; a key the file holds that heddle never set stays.

### `lym-no-attribution`

- From: shipped with lym
- Order: 100
- Targets: `claude-settings`

```json
{
  "attribution": {
    "commit": "",
    "pr": ""
  }
}
```

### `lym-plugins`

- From: shipped with lym
- Order: 110
- Targets: `claude-settings`

```json
{
  "enabledPlugins": {
    "ly@loomyard": true,
    "scribe@scribe": true
  },
  "extraKnownMarketplaces": {
    "scribe": {
      "source": {
        "repo": "Knatte18/scribe",
        "source": "github"
      }
    }
  }
}
```

### `lym-tool-deny`

- From: shipped with lym
- Order: 120
- Targets: `claude-settings`

```json
{
  "permissions": {
    "deny": [
      "Artifact",
      "Workflow",
      "ReportFindings",
      "ScheduleWakeup",
      "SendFeedback",
      "CronCreate",
      "CronDelete",
      "CronList",
      "RemoteTrigger",
      "DesignSync",
      "NotebookEdit",
      "EnterWorktree",
      "ExitWorktree",
      "Skill(dataviz)"
    ]
  }
}
```

### `lym-session-scope`

- From: shipped with lym
- Order: 130
- Targets: `claude-settings`

```json
{
  "disableBundledSkills": true,
  "disableClaudeAiConnectors": true,
  "disableWorkflows": true
}
```

### `lym-prompt-suggestion-off`

- From: shipped with lym
- Order: 140
- Targets: `claude-settings`

```json
{
  "promptSuggestionEnabled": false
}
```

### `lym-agent-defaults`

- From: derived from the hub's agents.yaml
- Order: 200
- Targets: `claude-settings`

```json
{
  "effortLevel": "medium",
  "env": {
    "CLAUDE_CODE_SUBAGENT_EFFORT": "medium"
  }
}
```

## `claude-prompt`

Appended to the system prompt of every claude session.

No entry.

## `codex-prompt`

Appended to the system prompt of every codex session.

No entry.

## `orch-prompt`

Appended to the system prompt of every orch session.

### `orch-rules`

- From: the hub's heddle.json
- Order: 1000
- Targets: `orch-prompt`

```markdown
# Orch rules for the loomyard hub

## Production

`update-plugins.sh` (`.cmd` on Windows) is the only route to production: never run it unasked, and never install `lym` anywhere else.
The one exception: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

## Pushing `main` by hand

A session that pushes `main` by hand pushes only from the worktree checked out on `main`, never from a task worktree.
It pushes only after `lym gate test --upto 2` is green on `main`, run as its own Bash call as the gate rule in the session's system prompt says; code under `tools/` counts like any other code.
A commit that changes only text no build or test reads, such as `docs/`, is pushed without the test run.

## Name entries by what they do

To the operator, explain an entry by what it does, never by its slug or handle alone: the operator does not keep slugs in mind, and a handle is reused once its entry lands (operator, 2026-11-01).

## A tapestry-sized note goes through Motif and raddle

A board note weighed for a tapestry run runs as a Motif first, and raddle then splits its Sketch, never as a tapestry task straight from the board (operator, 2026-11-08).
Raddle chooses the shape of each part, tapestry included, and that choice stands.
```
