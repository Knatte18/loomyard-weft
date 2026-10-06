---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-drop-per-skill-load.md Intent, and 02-launch-skills-one-turn.md and 03-orch-reload-one-turn.md Intent
    summary: Card 4 removes the pane unknown-skill classifier, and cards 2 and 3 add the unverified pass-through, without stating in Intent that an unknown skill on an unreadable-transcript turn goes undetected and what bounds that
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/session.go:151 via _lyx/plan/04-drop-per-skill-load.md
    summary: The defaultSkillLoadTimeout comment says it bounds one skill's load, which the change falsifies, and no card edits it
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/01-skill-load-transcript.md Intent (claudeengine/doc.go rewrite)
    summary: Card 1 rewrites the claudeengine skill-loading doc paragraph to the new protocol and drops the /<skill> line, while production still uses the old path until cards 2 to 4, so the doc is false at card 1's commit
---

### [MEDIUM] Removed and downgraded skill-load guards carry no stated bound in Intent

**Location:** `_lyx/plan/04-drop-per-skill-load.md` Intent; `02-launch-skills-one-turn.md` Intent (the `Verified` false bullet); `03-orch-reload-one-turn.md` Intent (the unverified report in the skills step).
**Issue:** The rubric's attack-surface question asks every card that removes or downgrades a guard to state in `**Intent:**` what it can now let through and what bounds it.
Decision 8 and Decision 6 each carry that bound in the decision record: with no readable transcript, an unknown skill is no longer detected and appears only in the `skill load unverified` Warn's list, with no stall.
Card 4 states only the removal ("Remove the pane-text unknown-skill classifier ... now that no caller uses them") and never says what the removal lets through.
Cards 2 and 3 say an unverified turn logs one Warn and moves on, but not that this is where an unknown skill goes undetected, nor that the Warn list is the only trace.
**Fix:** Add one sentence to card 4's Intent stating that after the removal an unknown skill on a turn whose transcript is unreadable is no longer detected and shows only in the `skill load unverified` Warn's list, with no stall (Decision 8's bound).
Add the same bound to the unverified bullet of cards 2 and 3.

### [LOW] `defaultSkillLoadTimeout` comment still says it bounds one skill's load

**Location:** `internal/shuttleengine/claudeengine/session.go:151`; plan card `04-drop-per-skill-load.md` (Edit `claudeengine/session.go`).
**Issue:** The comment reads "bounds one skill's load: a skill load is one short model turn".
Decision 7 makes the timeout bound one load turn for the whole list, so the comment is false after card 2.
Card 4 already edits `claudeengine/session.go` but names only its file comment.
No other card touches the constant's comment, and the rubric's consistency class covers a comment that departs from the change.
**Fix:** In card 4's `claudeengine/session.go` edit, also reword the `defaultSkillLoadTimeout` comment to say it bounds one load turn of the whole skill list.

### [LOW] Card 1 rewrites the claudeengine doc ahead of the code it describes

**Location:** `_lyx/plan/01-skill-load-transcript.md` Intent, last paragraph.
**Issue:** Card 1 rewrites `claudeengine/doc.go`'s skill-loading paragraph to describe only the one typed message and the transcript check, "dropping the `/<skill>` line and the unknown-command notice".
The same card says the old `/<skill>` path "keeps working until card 4", and shuttle's launch and the orch reload use it until cards 2 and 3.
The decision record's Constraints require docs to land in the same commit as the change they describe, so the doc is wrong at the commit of card 1, and again in `shuttleengine/doc.go` terms until card 3.
**Fix:** Have card 1 add the new paragraph beside the old one and say the per-skill path remains until it is replaced, and have card 4 remove the old text from `claudeengine/doc.go` in the commit that deletes `SkillLoadSequence` and `SkillUnknown`.
Card 4 then lists `claudeengine/doc.go` under `Edit`.

## Focus verdicts

These settle the directive's questions; none departs from the directive.

- **Fidelity, cards 2 and 3 against Decisions 5, 6, 7, 10, 11, 12:** not a defect.
  Both cards carry one retry, the retry classified with the same three classes, no retry on timeout or unverified, the offset moving past the retry's turn end, the persisted retry list written with the step in one save, and the old `reload_step` read as the pointer step.
  Card 3's retry value (`-1`, no old index can take it) satisfies Decision 12's collision clause.
- **Fidelity, Decision 14 and the two stubs:** not a defect.
  The stubs' change is confined to the skill-load answer and its comment, the settings file and `events.jsonl` both sit in the run directory (`settingsFileName`, `eventsFileName` in `run.go`), and card 2's `Edit` set adds nothing Decision 14 does not name.
- **Sequencing across cards 2, 3 and 4:** not a defect.
  The only production callers of `Runner.LoadSkill`, `Runner.SkillUnknown`, `SkillLoadSequence` and `SkillUnknown` are `run.go` (card 2), `orchengine/watcher.go` and `orchcli/session.go` (card 3) and the two `Runner` methods themselves, plus the fakes and tests named in cards 2 and 4.
  The tree builds after each card, and card 4's `grep` comes back empty once its fake and doc edits land.
- **Verify coverage:** not a defect.
  Every package a card targets (`shuttleengine`, `claudeengine`, `shuttlefake`, `orchengine`, `orchcli`, `loomcli`) sits under a `./...` pattern run untagged, with `integration` and `tmux`, and vetted with `llm`.
  The `tmux` stubs are in `loomcli`, which the list includes.
- **Attack-surface question:** the one gap is F1.
  The retry step's routing edge is bounded in card 3 (idle-gated, never retried twice), and card 2 bounds the number of turns.
- **Test economy:** not a defect.
  Card 2 extends `TestStartup_SkillsLoadInOrderBeforePrompt` and `TestStartup_SkillSkipCauses`, and card 3 turns `TestWatcher_SkillSkipCauses` and the reload restart tests into rows, rather than adding parallel tests.
  Card 1's new test file covers a new file's public surface, which no existing test covers.
