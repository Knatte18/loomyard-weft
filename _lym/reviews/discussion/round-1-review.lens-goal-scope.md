```yaml
lens: goal-scope
outcome: reported
```

Lens: goal-scope. I checked the record against the `creel-hook-post` board entry and looked for this task's decision notes.

**Goal points of the entry, and whether the record meets them**

- **Goal: hooks deliver in full, visibly, as fast as possible; a mid-turn scope change lands at the next tool call.** Met by the record's Goal and D1.
- **Codex is out until `codexengine` exists.** Met: Scope Out names `codex-engine`.
- **Evidence: use `block` with `reason` and `systemMessage`; `additionalContext` is out.** Met by D1.
- **While the receiver works, `PostToolUse` blocks with `reason` and `systemMessage`.** Met by D1.
- **After the last tool call, `Stop` blocks with `reason` only while unread mail stands.** Met by D2. Its cap of 6 adds a stop condition but does not go against the entry.
- **While idle, an `asyncRewake` wakes with the lines, the text is visible, and the build stops and asks the parent when it finds no way.** Partly met. D4 wakes with a fixed bell, and the mail shows in the next `Stop` block. The record asks the parent under Open risks (line 1), as the entry directs, so I raise no finding.
- **Each message is delivered once, by whichever path reserves it first, and a waiter that loses the race delivers nothing.** Met: D4's waiter claims nothing, and an acceptance criterion covers the race.
- **Main thread only, filtered on `agent_id`, "to be confirmed by a test".** Met by D3, by the Constraints line saying the LLM-tier test pins it, and by an acceptance criterion with a real fork.
- **Pause and hold keep their rules.** Met: D4/D5 and the acceptance criteria apply them.
- **The 8 KiB claim budget stays.** Met by D1.
- **Agent mail goes in full, and the reminder and its `lym creel read` turn go.** Met by D6.
- **The pane path for posts goes; a session launched before the deploy gets posts once it is restarted.** Partly met. D5 types a bell into the pane, which the record asks the parent about (Open risks, line 2). D5 also bells pre-deploy sessions, which goes against "once it is restarted": see F1.
- **`inject` keeps typing.** Met: Scope Out.
- **`post --interrupt` sends the interrupt, then posts, and the message reaches the stopped session through the idle wake.** Met by D7, with the order validate, interrupt, store. The route through D5's bell is raised under Open risks, line 2.
- **Quote strand names in `MessageLine` and in `lym creel read`.** Met by D8, which quotes every sender; that choice is recorded as an auto-pick.
- **The trust sentence goes verbatim via `--append-system-prompt` on every Claude session.** Met: D9's text matches the entry word for word.
- **Cobra help on the `creel` group and on `post` carries the reply rule.** Met by D10.
- **Tests run through shuttle's rendered `settings.json` and launch line, covering mid-turn delivery, the `Stop` block, the idle wake and its text, the fork/subagent filter, and `post --interrupt`.** Met by the acceptance criteria and their header line.

**Decision notes:** `lym board list` names no `Decision for creel-hook-post:` note, so I read none and there was none I could not reach. The record has no `Added after Discussion` entries.

**Findings**

- **F1**
  - Severity: MEDIUM. Class: `scope`.
  - Location: `decision-record.md` D5 and Open risks, line 3.
  - Summary: D5 types the bell into sessions launched before the deploy, against the entry's "a session launched before the deploy gets posts once it is restarted", and there the bell's promise is false.
  - Why the promise is false: such a session's `Stop` hook is only the recording hook, with no creel delivery and no waiter. The bell tells the agent to "End this turn now; creel hands the mail to you as the turn ends. Do not run lym creel read for it." The turn ends with no delivery, and D5 never repeats the bell for an unchanged set. The mail then waits for the session's next tool call in some later turn.
  - The record states this only as a risk line ("D5 may bell them once per claimable set"). It is neither a question for the parent nor a decision, and the writer can settle it, for example by limiting D5 to sessions whose rendered settings carry the new `Stop` delivery, or by stating what the bell does there.
  - Evidence:
    - The entry body, `## Creel changes`, second bullet ("a session launched before the deploy gets posts once it is restarted").
    - `decision-record.md` D5 (bell conditions; "Only a strand whose session shuttle launched gets it") and D4's bell text.
    - Open risks, line 3.
    - `internal/shuttleengine/claudeengine/settings.go:314`, where the current `Stop` entry is `recordingHook(hookEventStop, false)` only.

**Tool calls**

1. `lym board list`, filtered on creel and "Decision for", to find this task's decision notes and touching entries. My context held only this task's body, which the parent fetched; it did not hold the list.
