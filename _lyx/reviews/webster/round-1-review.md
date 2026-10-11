---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/send.go:507
    summary: confirmPasted waits out the whole submit window on /clear and /compact for an engine without SessionSignalParser, which breaks the plan's no-capability-unchanged decision
  - id: F2
    severity: LOW
    class: scope
    location: internal/shuttleengine/session_test.go:325
    summary: no test row has a matching session-start line before the pre-send offset, the stale-line case card 6 exists to reject
  - id: F3
    severity: LOW
    class: scope
    location: internal/shuttleengine/claudeengine/activity_test.go:101
    summary: the backward widening in holdsPendingToolCall (message cut by the window edge) is not exercised by any test
---

Range: `main`'s merge base to `HEAD` (`status.json` `product.parent` is `main`, no `_lyx/loom/rework`), eleven commits, one per card.
`lyx gate test` over `internal/shuttleengine/...`, `internal/battenshed`, `internal/orchengine` and `cmd/lyx` (test budget included) passes, and `lyx loom lint-comments` is clean.
I ran no other tier.

Focus questions, settled:

- **Provider seam:** clean.
  Hook names, payload fields, source strings, notification types and the stray-character judgment sit in `claudeengine`.
  `shuttleengine` sees `SessionStartSource`, `SessionSignalKind` and `StrayInput` readings only.
  A grep of the non-test added lines in `shuttleengine` found no provider specifics.
- **Card 7 gating:** met.
  The corpus case `hook-capture/case.yaml` records `SessionStart(startup)` first in file order, before the first `UserPromptSubmit`, and records that the resumed launch types no turn and still writes `SessionStart(resume)`.
  The diff builds card 7 (`RecordsSessionStarts` on `Launch` and `RunState`, `sessionStartRecorded`, `checkLivenessTick`, `abandonStartup`), which matches that outcome.
- **Card 6 offsets:** `sessionStartedSince` parses `data[offset:]` and compares the exact source, so a stale line before the offset and a mismatched source never confirm.
  The code is right; only the stale-line case lacks a test (F2).
- **Card 10 clear:** at most once, since `awaitIdleSession` creates one `strayClear` and the recursion in `clearStrayCharacter` runs with `cleared` set.
  The clear sits behind the hook reading held only for a draft, or the pane reading ready with no unmatched turn start, behind `StrayInput`'s no-running-turn check, and behind `operatorActivityHold`.
  The character is read as residue only on post-clear readings and only when it equals the recorded one.
  Not a defect.
- **Card 5 tail:** the pending call comes from the newest tool-calling message only.
  A window cut mid-line drops the first partial line in `readBackward`, a partial last line fails to unmarshal and is skipped, and a message cut at the window edge is not `settled`, so the window widens.
  A transcript with no tool use reports false after a read back to the file start.
  Not a defect (F3 is a coverage gap only).
- **Card 8 death:** the registry alone ends a run only for an id this wait read alive, and a `SessionEnd` ends one only if it is the newest signal past the first-tick size and the registry does not read alive.
  `processLiveness` answers dead only from an entry that names the id with a gone pid, so a `/clear` that rewrites the entry to a new id reads unproven, not dead, for the old id.
  I found no path that reads a live process as gone.
- **Card 11, docs, budget:** the notice change, the `docs/overview.md` row and the `PATTERN-pane-text-injection` background match their cards.
  The `claudeengine` and `shuttlecli` budget rows are raised by exactly the one new top-level test each.
  `TestStartup_NeedsSessionStartLine` is a new table-driven test; `TestStartup_TrustPromptThenReady` is a single scenario, so the table does not fit there and I raise no economy finding.

### [MEDIUM] No-capability engines wait out the submit window on /clear and /compact

**Location:** `internal/shuttleengine/send.go:507`
**Issue:** `awaitsLanding` is `sc.landingStart != ""`, and `ClearSession` and `CompactSession` set it for every engine.
For an engine without `SessionSignalParser`, `sessionStartedSince` can never answer true, so an empty box no longer ends the reads and the send blocks for the full `submitConfirmTimeout` before confirming.
The plan's Shared Decisions say an engine without `SessionSignalParser` behaves exactly as today on every path the plan touches.
Card 6's own text accepts the fallback ("falls back to the box read at its end"), so the card and the shared decision disagree, and the diff follows the card.
The test row "no session start confirms on the empty box once the window closes" pins the slow behavior instead of the old one.
**Fix:** set `awaitsLanding` only when `sc.engine` implements `SessionSignalParser`, so a parser-less engine confirms on the empty box at once as before.
Keep the row for an engine that parses signals and records none.

### [LOW] Stale matching session-start line is untested

**Location:** `internal/shuttleengine/session_test.go:325`
**Issue:** `checkLandingStartConfirmation` appends the line on the paste, so every row has the line after the offset.
Card 6's point is "past the pre-send offset", and no row seeds the events file with a matching `SESSIONSTART:<source>` line before the send and expects it not to confirm.
`newSlashPasteRunnerAppending` writes only `START` and `STOP:done`.
**Fix:** add a row to the same table whose events file already holds a line of the landing source, with no paste line, expecting the full-window wait.

### [LOW] Window widening in `holdsPendingToolCall` is untested

**Location:** `internal/shuttleengine/claudeengine/activity_test.go:101`
**Issue:** the new rows run in one window (`initialReadChunk` exceeds their size), so the `settled == false` path that doubles the window until the message's older entries and the next older assistant entry are in view never runs.
A regression that returns `settled` early would pass every row.
**Fix:** add a row, or a row-driven variant, whose transcript is larger than `initialReadChunk` with a pending call in the first entry of a parallel batch that lies before the first window's start.

