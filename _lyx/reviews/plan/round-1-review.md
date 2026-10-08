---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/00-overview.md (Card Index, no card)
    summary: "No card carries the two decisions added after Discussion that route the segment color through the send path; Start's playInputs(ColorSequence) and Runner.TypeColor stay raw"
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/10-delete-strand-removers.md
    summary: "The removers cards 8 and 9 orphan are deleted only by card 10, so card 9 lands PATTERN-shuttle-stop while burler and webster removers still exist"
  - id: F3
    severity: NIT
    class: decision
    location: _lyx/plan/01-shuttle-send-config.md, _lyx/plan/05-start-send-teardown.md, _lyx/plan/08-burler-stops-through-handle.md
    summary: "Three cards hedge which test to extend or whether to add a row instead of naming it"
---

### [BLOCKING] The segment-color decisions are carried by no card

**Location:** `_lyx/plan/00-overview.md` Card Index; the cards touching `internal/shuttleengine#Run.loadSkillsThenPrompt`, `Runner.TypeColor` and start (none lists them for color).
**Issue:** The decision record has two entries added after Discussion, "The segment color goes through the send path after readiness" and "Burler: the merged raw /color typing is a blocking finding".
Together they decide that every path that types the segment color into a session (start's `playInputs(…, ColorSequence(…))` after `awaitStartup`, and `Runner.TypeColor` for a caller that colors itself, such as the loom driver and the orch watcher) plays only after the provider is ready and goes through this task's send discipline (idle wait, settled box, confirmed landing), with a failed color send logged and never failing the launch.
The record says the task owns this fix and that three named `loomcli` tmux-tier tests must pass.
No card in the plan mentions color: `grep -i color _lyx/plan` finds nothing.
Start's color play in `internal/shuttleengine/run.go` (`playInputs(r.reed, strand.GUID, r.engine.ColorSequence(strand.Color))`) and `Runner.TypeColor` in `internal/shuttleengine/session.go` (`playInputs(r.reed, guid, r.engine.ColorSequence(color))`) are both still raw typing, so executing the plan leaves exactly the defect the record names, and the landing and idle disciplines of cards 3 and 4 never reach the color command.
**Fix:** Add a card after card 4 (it needs `sendVerified`'s `sendContext`, idle wait and landing confirmation) and before card 5 or after it, since start's call sits between `awaitStartup` and `loadSkillsThenPrompt`.
It `Edit`s `internal/shuttleengine#Run.awaitStartup` or the start function that plays the color, and `internal/shuttleengine#Runner.TypeColor`, routes both through the verified send, logs a failed send at Warn without failing the launch, and carries the test rows for both paths plus an `ImpactSummary` naming the callers of `TypeColor` (the orch watcher's session and the loom driver).
Its `**Intent:**` states the bound the attack-surface rule needs: a color send can only wait or fail and is display-only.

### [LOW] Dead removers survive two commits and make PATTERN-shuttle-stop false when it lands

**Location:** `_lyx/plan/10-delete-strand-removers.md`, `_lyx/plan/09-webster-stops-through-shuttle.md`.
**Issue:** Card 8 drops every caller of `burlerengine.StrandRemover`, `reedStrandRemover`, `NewReedStrandRemover` and `shedfake.StrandRemover`, and card 9 drops every caller of `websterengine.removeStrandIfLive`; card 10 then deletes them.
A deletion that belongs to the card that orphans the symbol is not a separate reviewable unit, and until card 10 each of those commits carries unused code.
Card 9 also adds `PATTERN-shuttle-stop`, whose text says no other package removes a shuttle run's strand, while `burlerengine/remover.go` and `removeStrandIfLive` still exist at that commit.
**Fix:** Move the burler and `shedfake.StrandRemover` deletions into card 8 and `removeStrandIfLive` with its test into card 9, and drop card 10; or keep card 10 and move the `PATTERN.md` entry into it.

### [NIT] Open hedges in test targets

**Location:** `_lyx/plan/01-shuttle-send-config.md`, `_lyx/plan/05-start-send-teardown.md`, `_lyx/plan/08-burler-stops-through-handle.md`.
**Issue:** Card 1 extends `TestLoadConfig_SubmitSettleKeys` "or the test that already tables per-key refusals", card 5 extends `TestStartup_SkillLoadTurns` "or `TestStartup_RunGatedMappings`, whichever tables start's outcomes", and card 8 "keeps (or gains, if no row pins it)" the attempt-2 transient-class row.
All the named tests exist, so the implementer is left a choice the plan can make.
**Fix:** Name the one test per card; for card 8 state whether `burler_test.go` already pins the row or add it as a stated new row.
