```yaml
lens: goal-scope
outcome: reported
```

Lens goal-scope: checking the record against the board entry `creel-hook-post` and any `Decision for creel-hook-post:` notes.

**Findings**

1. **MEDIUM**, class `decision`, D4 and Open risks: the idle wake no longer carries the message.
   - Summary: the entry says the `asyncRewake` hook "wakes the session with the lines", the build makes that text visible, and the build "stops and asks the parent when it finds no way". The record finds no way to show rewake text, so the wake carries a fixed bell and the next Stop block carries the mail.
   - Evidence: entry, "Delivery" bullet 3; record, D4 and its Constraints line on `rewakeSummary`.
   - Question for the parent: the entry's no-way case calls for asking, not for picking a substitute. Is a fixed-bell wake followed by a visible Stop-block delivery, at the cost of one extra model turn per idle wake, an acceptable answer to "make the message text itself visible"?

2. **MEDIUM**, class `decision`, D5: a typed bell keeps a pane path the entry removes.
   - Summary: the entry says "The pane path for posts goes: every session is launched by shuttle and has the hooks". D5 still types a fixed bell into an idle strand that has no live waiter.
   - Evidence: entry, "Creel changes" bullet 2; record, D5 and Open risks bullet 1.
   - Question for the parent: is a typed bell with no sender text acceptable? The alternative is to drop D5 and accept that mail to an interrupted, cleared or day-idle session waits for its next turn.
   - This also weakens D7. The entry says `post --interrupt` reaches the stopped session "at once through the idle wake". Since no `Stop` fires after a user interrupt, D7 will mostly run through D5's typed bell, not through the waiter.

3. **LOW**, class `scope`, Acceptance criteria: the "through shuttle" qualifier is missing on three scenarios.
   - Summary: the entry requires every one of the five scenarios to be tested through shuttle's rendered `settings.json` and launch arguments, never with hand-launched sessions. The record says so only for mid-turn delivery and the fork filter. The Stop block, the idle wake with its visible text, and `post --interrupt` carry no such qualifier.
   - Evidence: entry, "Test" section; record, Acceptance criteria bullets 1, 2, 4, 5 and 7–10.
   - Fix: state that all five scenarios run in the LLM tier with real Claude through shuttle's rendered settings and launch line.

4. **LOW**, class `scope`, D8: the quoting reaches past what the entry names.
   - Summary: the entry says to quote "wherever creel writes a strand name in text" and names `MessageLine` and `lym creel read`. D8 also quotes program names and `EXTERNAL`, which is not a strand name.
   - Evidence: entry, "Creel changes" last bullet; record, D8 and Auto-mode assumption 4.
   - The extension is harmless and already recorded as an auto-pick. Keep it, or narrow it to strand and program names.

**Goal points of the entry**

| Goal point | Met by | Verdict |
|---|---|---|
| `PostToolUse` delivery as block with `reason` and `systemMessage`, visible under the tool result | D1 | met |
| `Stop` block for mail after the last tool call, only while unread mail stands, so no loop | D2 | met |
| `asyncRewake` wakes an idle session with the lines, text made visible, parent asked if no way | D4 | partly met: finding 1 |
| Delivered once by whichever path reserves first; a waiter that loses the race delivers nothing | D4 (the waiter claims nothing), D6 | met |
| Main thread only, by `agent_id`, confirmed by a test | D3, Constraints, Acceptance criteria | met |
| Pause and hold rules kept | D4, D5, Acceptance criteria | met |
| 8 KiB budget kept, the rest at the next boundary | D1, Acceptance criteria | met |
| Agent mail in full; the reminder and its `lym creel read` turn go | D6 | met |
| Pane path for posts goes | D6 | met, except D5: finding 2 |
| Pre-deploy sessions get posts once restarted | Scope (out), Open risks | met |
| `inject` keeps typing | Scope | met |
| `post --interrupt` sends the guarded interrupt, then posts; `interrupt` stays a verb | D7 | met; the "at once through the idle wake" path is weaker: finding 2 |
| Strand names quoted in `MessageLine` and `read` output | D8 | met: finding 4 |
| Trust and reply sentence through `--append-system-prompt` on every Claude session | D9 | met; verbatim, on launch and resume lines |
| Cobra help on the `creel` group and on `post` carries the reply rule | D10 | met |
| Tests through shuttle's rendered settings and launch arguments, five scenarios | Acceptance criteria | partly met: finding 3 |
| Codex out of scope | Scope | met |

No work belongs to another entry: `creel-agents` and `codex-engine` stay out. Nothing is deferred. D2's merged Stop command and D11's stencil and doc updates follow from the entry and `CLAUDE.md`, so they are licensed.

**Decision notes:** `lym board list`, already in my context, shows no note titled `Decision for creel-hook-post:`. None was read and none was unreachable. The record has no `Added after Discussion` entries.

**Tool calls:** none. The board list and the task's board entry were already in my context.
