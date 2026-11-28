# Heddle

What heddle delivers to every worktree and session in this hub.
lym rewrites this file whole at every heddle write; change an entry with `lym heddle set`, never this file.
Two targets are shared: `agents-file` lands in both agent files, and `agents-prompt` in the prompt of every Claude and Codex session.

Sources:

- **repo** — this hub's `heddle.json`; yours to change.
- **lym** — shipped with lym; change it only through an override (`lym heddle set --layer lym`).
- **lym, overridden** — shipped with lym, its text replaced by this hub's override.
- **derived** — rendered from this hub's config, such as `agents.yaml`.

## At a glance

| Lands in | Source | Entry | What it does |
|---|---|---|---|
| `CLAUDE.local.md` | lym | `lym-batching` | Batch your calls |
| `CLAUDE.local.md` | lym | `lym-plan-edits-before-gate` | Make every round of edits in this order |
| `CLAUDE.local.md` | lym | `lym-shell-directory` | Keep the shell where it started |
| `CLAUDE.local.md` | **repo** | `nicknames` | Nicknames |
| `AGENTS.override.md` | lym | `lym-batching` | Batch your calls |
| `AGENTS.override.md` | lym | `lym-plan-edits-before-gate` | Make every round of edits in this order |
| `AGENTS.override.md` | lym | `lym-shell-directory` | Keep the shell where it started |
| `.claude/settings.local.json` | lym | `lym-no-attribution` | `attribution` |
| `.claude/settings.local.json` | lym | `lym-plugins` | `enabledPlugins`, `extraKnownMarketplaces` |
| `.claude/settings.local.json` | lym | `lym-tool-deny` | `permissions` |
| `.claude/settings.local.json` | lym | `lym-session-scope` | `disableBundledSkills`, `disableClaudeAiConnectors`, `disableWorkflows` |
| `.claude/settings.local.json` | lym | `lym-prompt-suggestion-off` | `promptSuggestionEnabled` |
| `.claude/settings.local.json` | derived | `lym-agent-defaults` | `effortLevel`, `env` |
| System prompt of every orch session | lym | `lym-glossary` | lym's glossary |
| System prompt of every orch session | **repo** | `orch-rules` | Orch rules for the loomyard hub |

## `CLAUDE.local.md`

Rendered at every worktree's anchor.

### From this repository

#### `nicknames`

```markdown
## Nicknames

- Read "Sketch" (capital S) as the decision record: the approved record a Discussion writes, stored on the board for a discussed entry.
- Read "Merriam" as webster's long-lived master session, which forks one implementer per batch.
- Read "jacq" as the jacquard recipe.
- Use every nickname in this list freely with the operator.
- Keep the formal names in code, files, verbs and specs.
```

### From lym

<details>
<summary><code>lym-batching</code> — Batch your calls</summary>

```markdown
## Batch your calls

- Request every call whose arguments you already know in one message.
- Before you look anything up, list every question the step raises.
- Send every grep and read for all of those questions in one message.
- Wait for a result only to choose the next call.
- When a result arrives, send every call it unlocks and every call you already know in one message.
- Read what you need before you edit: a file whole, or every span you need from it, in one message.
- Read a file whole, or every span you need from it, even when your read tool advises reading only the part you need.
- Never grep for a line number and then read a slice around it.
- Never read one file in slices over several turns.
- Keep a file you have read in mind; read it again only when something other than your own edit has changed it.
- After a command rewrites files, such as a formatter, a generator or a script, read every file it changed again in one message before you edit any of them.
```

</details>

<details>
<summary><code>lym-plan-edits-before-gate</code> — Make every round of edits in this order</summary>

```markdown
## Make every round of edits in this order

- Follow the steps below in order every round, and never deviate from them.
- Count as a round any set of edits, whatever starts it: a card, a task, a plan, a review's findings, a fixer's report or a failed gate.

1. Read everything the round needs in one message: every read and search together, a file whole when it fits and every span of it you need otherwise.
   - Make a second round of reads only for files the first round named.
2. Decide every edit across every file the round touches before you make any of them.
3. Make all of those edits in one message, every file edit together in one turn.
4. Run the round's gate once, or its validator when it has no gate, and fix everything its report shows the same way:
   1. Read what the fixes need in one message.
   2. Decide every fix.
   3. Make all of them in one message.
   4. Run the gate or validator again.

- Never read one file per turn.
- Never spread an edit, a read and another edit over turns.
- Never run the gate between the edits of one round.
```

</details>

<details>
<summary><code>lym-shell-directory</code> — Keep the shell where it started</summary>

```markdown
## Keep the shell where it started

- The shell keeps the directory a `cd` moves it to for every later command, so a `lym` verb run after the `cd` may be refused, far from the `cd` that caused it.
- Reach another directory through an absolute path, `git -C <dir>` or `env -C <dir>` instead of `cd`.
```

</details>

## `AGENTS.override.md`

Rendered at every worktree's anchor.

### From lym

<details>
<summary><code>lym-batching</code> — Batch your calls</summary>

```markdown
## Batch your calls

- Request every call whose arguments you already know in one message.
- Before you look anything up, list every question the step raises.
- Send every grep and read for all of those questions in one message.
- Wait for a result only to choose the next call.
- When a result arrives, send every call it unlocks and every call you already know in one message.
- Read what you need before you edit: a file whole, or every span you need from it, in one message.
- Read a file whole, or every span you need from it, even when your read tool advises reading only the part you need.
- Never grep for a line number and then read a slice around it.
- Never read one file in slices over several turns.
- Keep a file you have read in mind; read it again only when something other than your own edit has changed it.
- After a command rewrites files, such as a formatter, a generator or a script, read every file it changed again in one message before you edit any of them.
```

</details>

<details>
<summary><code>lym-plan-edits-before-gate</code> — Make every round of edits in this order</summary>

```markdown
## Make every round of edits in this order

- Follow the steps below in order every round, and never deviate from them.
- Count as a round any set of edits, whatever starts it: a card, a task, a plan, a review's findings, a fixer's report or a failed gate.

1. Read everything the round needs in one message: every read and search together, a file whole when it fits and every span of it you need otherwise.
   - Make a second round of reads only for files the first round named.
2. Decide every edit across every file the round touches before you make any of them.
3. Make all of those edits in one message, every file edit together in one turn.
4. Run the round's gate once, or its validator when it has no gate, and fix everything its report shows the same way:
   1. Read what the fixes need in one message.
   2. Decide every fix.
   3. Make all of them in one message.
   4. Run the gate or validator again.

- Never read one file per turn.
- Never spread an edit, a read and another edit over turns.
- Never run the gate between the edits of one round.
```

</details>

<details>
<summary><code>lym-shell-directory</code> — Keep the shell where it started</summary>

```markdown
## Keep the shell where it started

- The shell keeps the directory a `cd` moves it to for every later command, so a `lym` verb run after the `cd` may be refused, far from the `cd` that caused it.
- Reach another directory through an absolute path, `git -C <dir>` or `env -C <dir>` instead of `cd`.
```

</details>

## `.claude/settings.local.json`

Merged into the file at every worktree's anchor; a key the file holds that heddle never set stays.
What heddle merges in, with the entry that sets each key:

```jsonc
{
  // lym-no-attribution (lym)
  "attribution": {
    "commit": "",
    "pr": ""
  },

  // lym-plugins (lym)
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
  },

  // lym-tool-deny (lym)
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
  },

  // lym-session-scope (lym)
  "disableBundledSkills": true,
  "disableClaudeAiConnectors": true,
  "disableWorkflows": true,

  // lym-prompt-suggestion-off (lym)
  "promptSuggestionEnabled": false,

  // lym-agent-defaults (derived)
  "effortLevel": "medium",
  "env": {
    "CLAUDE_CODE_SUBAGENT_EFFORT": "medium"
  }
}
```

## System prompt of every orch session

Appended to the system prompt of every orch session at its launch.

### From this repository

#### `orch-rules`

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
```

### From lym

<details>
<summary><code>lym-glossary</code> — lym&#39;s glossary</summary>

```markdown
## lym's glossary

Each line names a lym term, its meaning, and the words never to use for it.
Use these terms with the operator, on the board and in records.
When someone uses a word a line rules out, or a term in another sense than its line gives, name the term and its line.
A term settled that this list lacks is added here, in lym's repository at `internal/heddleengine/glossary.md`.

- **board**: the hub's list of entries. Never: backlog, tracker.
- **entry**: one item on the board, a task, a note or a project. Never: card, kort, ticket, issue.
- **task**: an entry a run works; only the operator makes or promotes one. Never: card, job.
- **note**: an entry that is not work yet, such as an idea, a finding or a Sketch. Never: card, draft.
- **project**: an entry that groups tasks under one aim. Never: epic.
- **decision record**: what a run's change must do, from Discussion or the entry. Never: log, logg.
- **support log**: Discussion's interview, rejected alternatives and review rounds, beside the decision record.
- **Sketch**: a decision record kept on the board as a note, for an entry discussed before it runs.
- **recipe**: the rows a run walks in order, such as tapestry, darn or sampler. Never: profile, profil, pipeline.
- **row**: one step of a recipe, run by one producer and checked by its gates. Never: stage.
- **run**: one recipe walked for one task in the task's worktree. Never: job, build.
- **plan**: the cards the Plan row writes for Webster to implement.
- **card**: one unit of a plan, which Webster implements in a batch. Never for an entry.
- **gate**: a mechanical check a row's output passes before the run moves on. Never: review.
- **change description**: the title and body a run lands under. Never: summary, log.
- **landing**: the merge of a run's task branch into its parent branch, in Finalize. Never: deploy, release.
- **hub**: one repository's worktrees, Bolt checkout, board and agents on a machine.
- **orch**: the operator's delegate session, which talks with the operator, writes the board and parents runs.
- **parent**: the session a run's agents ask and report to, the orch or a run above them.
- **autonomy**: a worktree's or a task's mode in which the orch settles every halt of the runs it covers and reaches the operator only for a severe halt, a merge-ready beam or a raddle split's approval. Never: autopilot, full auto.
- **severe halt**: a halt in one of autonomy's closed classes, which the orch relays to the operator. Never: critical halt, blocker.
- **strand**: one agent's pane, holding one session. Never: tab, window.
- **beam**: a long-lived branch that groups board work; its tasks fork from it and land back into it.
- **pair**: a task's code worktree together with its records worktree.
- **pusher**: the background process that brings one worktree's run commits to origin in order.
- **move note**: what a cancelled run learned, written for the fresh run that replaces it. Never: hand-back, handoff.
```

</details>
