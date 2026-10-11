---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/10-claude-paste-live-check.md
    summary: Card 10's llm-tier test in package shuttlecli sizes its texts by pasteCollapseMinChars, an unexported claudeengine constant card 9 creates, which shuttlecli cannot reference
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-shuttle-paste-path.md
    summary: Card 6's foreign-box failure is a plain ErrSessionBusy error that does not quote the box, yet card 13's driver_busy message and card 14's batten report rely on shuttle's reason quoting the draft (D8, D9)
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/08-shuttle-session-commands.md
    summary: Card 8 rewrites typeColor as sendSequence over ColorSequence without keeping the empty-sequence no-op, so a color with no command runs the idle wait and a verified send of empty text
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/03-reed-name-repair-guard.md
    summary: Card 3 adds a fail-open on an activity-query error for the name repair but its Intent states no bound for it
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/16-batten-status-halted-flag.md
    summary: Card 16 tests only the pure haltedOverDoneChild, leaving readChildStatus and the key battenStatusExtras emits, the surface D10's acceptance names, unpinned
---

### [MEDIUM] Card 10 references an unexported claudeengine constant from shuttlecli

**Location:** `_lyx/plan/10-claude-paste-live-check.md`, Intent, third bullet; the constant is created in `_lyx/plan/09-claude-paste-path.md` as `plan:internal/shuttleengine/claudeengine#pasteCollapseMinChars`.

**Issue:** `smoke_paste_test.go` lives in `internal/shuttlecli` (every sibling `smoke_*_test.go` there is in that package), and card 10 has it paste "a text of `pasteCollapseMinChars` characters" and one character shorter.
Card 9 declares `pasteCollapseMinChars` unexported in `claudeengine`, so the test cannot name it.
The plan's `## verify:` runs `go vet -tags llm ./...`, which type-checks this file, so as written the card fails at verify.
The implementer might also hard-code 800 in the test, which would let the test and the threshold drift apart.
That defeats the card's purpose: it pins the threshold claudeengine uses (D5's bound and A7).

**Fix:** Have card 9 expose the threshold to the test, for example through an exported `Claude` method or exported constant listed under card 9's `**Create:**`, and have card 10 list it under `**Uses:**` and size its texts from it.
Alternatively, move the threshold check into an `llm`-tier test inside `claudeengine`, which the verify section already compiles.

### [MEDIUM] The foreign-box failure does not carry the draft that the driver_busy refusal must name

**Location:** `_lyx/plan/06-shuttle-paste-path.md`, the `confirmPasted` bullet "any other text: return a plain error wrapping `ErrSessionBusy` (\"the input box holds text that is not this send's\"), with no key sent".

**Issue:** D8 requires the `driver_busy` refusal message to name "the reason from shuttle (including the draft text)".
The acceptance criteria require `lyx loom start`/`resume` to "name the draft", and D9 has batten's report name the draft.
Card 13's `driverBusyMessage` says shuttle's reason "quotes the draft", and card 14 says the child's refusal reason "quotes the draft".
The pre-write hold quotes the draft through `judgeReadiness`'s existing `the input box holds a draft: %q`, and card 6 makes `ErrSentTextInDraft` quote the box.
The third outcome, other text in the box after the paste-submit, is specified only by the fixed text above, with no box text and no pane tail.
A driver whose operator started typing in the gap between the guard and the paste therefore gets a `driver_busy` refusal that does not show the draft.
Cards 13 and 14 assume what card 6 does not deliver.

**Fix:** In card 6, make the foreign-box error quote the box it read, as the in-draft branch does (for example `the input box holds text that is not this send's: %q`).
Add to card 6's foreign-draft test row an assertion that the error text carries the draft.

### [LOW] Card 8 drops typeColor's empty-sequence no-op

**Location:** `_lyx/plan/08-shuttle-session-commands.md`, Intent: "`typeColor(sc, color)` becomes `sendSequence(sc, engine.ColorSequence(color))`".

**Issue:** Today `typeColor` returns nil at once when `ColorSequence` is empty (`internal/shuttleengine/run.go`, `if len(inputs) == 0 { return nil }`).
`Claude.ColorSequence` returns nil for a color with no name (`internal/shuttleengine/claudeengine/color.go`).
`Runner.TypeColor`'s contract is "a provider with no color command, or none for color, types nothing".
`sendSequence` as specified sends `sendVerified(sc, sequenceText(inputs))` on the paste path.
On any other path it runs `awaitIdleSession` then `playInputs`.
For an empty sequence the paste path makes a verified send of "": an idle wait, then a paste-submit of an empty buffer with an Enter.
The non-paste path makes start's color step and the orch watcher's color step wait for idleness for nothing.
The card states neither the no-op nor a test for it.

**Fix:** State in card 8 that `sendSequence` (or `typeColor`) returns nil before the guard when the sequence is empty.
Add a `TestRunner_TypeColor_PlaysTheSequenceAndWaitsItOut` row: a color with no command types nothing and makes no `PasteSubmit` or `OperatorActive` call.

### [LOW] Card 3's activity fail-open has no stated bound

**Location:** `_lyx/plan/03-reed-name-repair-guard.md`, Intent: "a query error logs at Warn and does not hold the rename (A4)".

**Issue:** The rubric's attack-surface rule asks the introducing card to state what a bypassable guard can now let through and what bounds it.
Card 5 does this for shuttle's fail-open: "the empty-box hold still stands".
Card 3 introduces the same fail-open into the watchdog's `/rename` path.
Its `Bound:` paragraph covers only the Enter variant, not what still stops a rename from merging with a draft when `list-clients` errors.
That is the `SessionNamer.SessionIdle` reading, which needs an idle pane with an empty box.

**Fix:** Add to card 3's Bound one sentence: on a query error the rename is still held by the namer's idle reading, which requires an empty input box, so a fail-open can type over at most an operator who is active but has not typed into the box yet (R2's window).

### [LOW] Card 16 pins the pure helper, not the status surface

**Location:** `_lyx/plan/16-batten-status-halted-flag.md`, Test paragraph.

**Issue:** D10's acceptance reads "`lyx batten status <slug>` for a failed, blocked or paused batten run whose child loom run is done carries the flag".
The card's only test is a table over `haltedOverDoneChild`, which takes `childFound` and the child status as arguments.
The parts that can go wrong are untested.
`readChildStatus` must resolve the pair through `taskWorktreeLocation` and read `shedrun.StatusFile(task, shedrun.SelfRunID)` under its lock.
It must answer found false for a missing pair or file.
`battenStatusExtras` must emit `childDoneFlagKey` only when the helper answers true.
A wrong run ID or path makes the flag never appear, while every planned row still passes.
`PATTERN-test-economy` asks for behavior pinned at the module's public surface.

**Fix:** Add a row, or extend an existing status test in `internal/battencli`, that writes a done child status file into a task-worktree fixture and asserts the status envelope carries `child_done_batten_halted` for a failed batten and omits it when the child status file is absent.
The pure table can then shrink to the state matrix alone.

## Focus items settled without a finding

- D3/D7: card 5 carries the Warn fail-open and states its bound. Card 11 carries the `StuckBySend` marker, which `probeIdle` and `logHeld` leave alone, as their code confirms. Card 11 also keeps a held step out of `maxConsecutiveTickErrors`.
- D5: card 6 replaces the needle and `isOwnText` and makes `clearUnlandedText` exact. Card 6 also adds the per-Enter activity check. Card 9 owns the collapse threshold and the line-count match. No D5 bound is missing.
- Scope: no card touches `Runner.Interrupt`, `Runner.Inject`, `InterruptSequence`, `ModelSwitchSequence`, `awaitStartup` or a queue module. No card's scope rests on the carry-over block.
- Card order: every `plan:` symbol is created at or before the card that uses it (cards 1→3/4, 5→6, 6→7/9, 12→13/15, 14→15).
- Re-sign arrows: every re-signed member carries one. No Edit card targets only non-test Go file paths while its Intent names symbols.
- Verify: `go test ./...` and `go test -tags integration ./...` cover every targeted package, including `contracts` and `cmd/lyx`. The `tmux` and `llm` tiers are compiled only, as the template states. The order matches the template.
- Test economy, cards 1, 2, 7 and 13: their tmux-tier and table rows cover behavior that no existing test covers.
- PATTERN.md: it is 14725 bytes, and card 9's entry line keeps it under `pattern.HeadroomBytes`.
