---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/09-webster-stops-through-shuttle.md
    summary: "The quoted PATTERN-shuttle-stop entry names only the loom driver exception, yet the card itself says orch keeps removing its strand raw, and the orch strand is a shuttle run"
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-color-through-send.md
    summary: "Start's color send for a launch whose prompt is on the command line waits out the running first turn, stalling Start for send_ready_timeout_s and then dropping the color; the Intent and Bound do not address it"
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/plan/05-start-send-teardown.md
    summary: "Several test-extension instructions leave the implementer a choice of test or table (cards 2, 5, 6, 8, 9, 11)"
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/11-color-through-send.md
    summary: "Card 11 keeps TestColorSequence_TypesTheColorCommandForEveryPaletteColor and its file header, which name ColorSequence after card 12 deletes it, and card 12 does not list color_test.go"
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/plan/03-send-waits-for-idle.md
    summary: "Card 11 builds its contexts with newSendContext on Run and Runner, which card 3 introduces but neither its Create list nor its Intent names"
---

### [MEDIUM] The entry's one exception does not cover the raw removals the plan leaves in place

**Location:** `_lyx/plan/09-webster-stops-through-shuttle.md` (Intent: "…the remaining raw `RemoveStrand` callers (reed's own verbs, orch, the loom driver) would make one a list of exceptions rather than a check").

**Issue:**
The card quotes the entry with exactly one exception, `loomcli`'s removal of the loom driver's strand.
The same card states that orch (and reed's own verbs) keep removing strands raw.
The decision record's Out list justifies orch with "no shuttle run record".
The code disagrees: `internal/orchcli/start.go` starts the orch session with `s.runner.Start(spec)`, and `Runner.start` calls `createRunDir` and `saveRunState`, so the orch strand has a run record.
`internal/orchcli/stop.go:41` and `internal/orchcli/start.go:225` then remove that strand through `reedStrandOps.RemoveStrand`, outside `internal/shuttleengine`.
The entry is therefore false on the day it lands for exactly the reason the decision record gives for the driver exception, and the plan carries no card, wording or note that resolves it.
The entry text is the answer key's, so the plan cannot reword it alone.

**Fix:**
The author decides one of: the entry names an orch exception as well; `orchcli`'s two removals move onto the stop verb (a scope the decision record excludes); or the card states why the orch strand is outside the entry and how the entry's wording says so.
Until that is decided the card should not claim the entry holds.

### [MEDIUM] The start-time color send blocks Start on a launch whose first turn is already running

**Location:** `_lyx/plan/11-color-through-send.md` (Intent, `Runner.start` paragraph and Bound).

**Issue:**
The card sends the color "after `awaitStartup` and before `loadSkillsThenPrompt`" under the unchanged condition `strand.Color != "" && !spec.ColorByCaller`, so it runs for every colored launch.
In `run.go` the prompt is deferred (`Launch.PromptLine`) only for a spec that names skills; any other spec carries the prompt pointer on the launch line, so the agent starts its first turn as soon as the provider is up.
Card 3's `awaitIdleSession` then holds the color send until the pane reads idle, i.e. until that first turn ends, for up to `send_ready_timeout_s` (60 s).
`Start` returns only after the send, so every skill-less colored launch stalls for the full window and then loses its color to `ErrSessionBusy`.
The card's Bound states the generic cap ("`send_ready_timeout_s` plus `submit_confirm_timeout_s`") but not that this case is the normal path for such launches, and no test row covers a launch without a deferred prompt.
Callers that start several runs in sequence (burler's two halves) pay the stall each time.

**Fix:**
State in the Intent how a launch with the prompt already on its command line is handled (for example the color is sent only when the prompt is deferred, or after the first turn end, or with an explicit shorter budget), and add the matching row to `TestRunner_Start_TypesTheSegmentColorAfterStartup`.
If the stall and the lost color are the accepted trade-off, say so in the Bound.

### [LOW] Test-extension instructions that leave a choice

**Location:** `_lyx/plan/02-claude-type-and-clear.md`, `05-start-send-teardown.md`, `06-gate-boundary-idle.md`, `08-burler-stops-through-handle.md`, `09-webster-stops-through-shuttle.md`, `11-color-through-send.md` (Tests paragraphs).

**Issue:**
The focus asked that every extended test be one named existing test, or an explicitly stated new row.
These do not meet it:

- Card 2: `session_test.go` "pins `ClearInputSequence` beside `TestClearSessionSequence`'s shape, and tables `PastePlaceholder`" does not say whether these are new functions or rows of the existing test.
- Card 5: "`RunGated`'s mapping row" names no test; the one that exists is `TestStartup_RunGatedMappings`, whose "mechanism failure returns the error with an empty outcome" row is the one the teardown changes, and its doc comment says the opposite.
- Card 6: "A row pins a re-prompt starting with less time left than its windows" names neither the table nor the test (`TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar` or the new `gate_test.go` table).
- Card 8: "`engine_test.go`'s round table" has two candidates, `TestEngine_Run_ShuttleOutcomes` and `TestEngine_Run_RoundFailureRules`; only the latter already carries the `NotStarted` rows.
- Card 9: "each websterengine test that reaches a reclaim" is a sweep, not a list; the tests that reach a reclaim are knowable (`runlevel_test.go`, `beginbatch_test.go`, `recoverbatch_test.go`, `strand_test.go`) and the card already lists those files, but not which test or table in each gets which assertion.
- Card 11: `TestRunner_TypeColor_PlaysTheEngineSequenceOnALiveShuttleStrand` is sequential assertions, not a table, so "gains a row where a busy pane fails `ErrSessionBusy`" has no row to join.

**Fix:**
Name the one existing test or table per instruction (for card 2, say new function or row; for card 5, `TestStartup_RunGatedMappings` and the row being reworded; for card 6, the table; for card 8, `TestEngine_Run_RoundFailureRules`; for card 9, one line per test file; for card 11, state a new assertion block or a restructure into a table).

### [NIT] Stale `ColorSequence` names in the color test after card 12

**Location:** `_lyx/plan/11-color-through-send.md` (Tests), `_lyx/plan/12-delete-color-sequence.md`.

**Issue:**
Card 11 repoints `claudeengine/color_test.go`'s `TestColorSequence_TypesTheColorCommandForEveryPaletteColor` at `ColorCommand` but keeps the name, and the file header (`color_test.go` line 1) names `ColorSequence`.
Card 12 deletes `ColorSequence` and its grep step rewords references "outside the deleted code", yet it lists neither the file nor the test.
`TestRunner_TypeColor_PlaysTheEngineSequenceOnALiveShuttleStrand` keeps a name ("Sequence") that no longer describes a key sequence.

**Fix:**
Have card 11 rename both tests and reword the header when it changes what they pin.

### [NIT] `newSendContext` is used by card 11 but named by no earlier card

**Location:** `_lyx/plan/03-send-waits-for-idle.md` (Create list, Intent), `_lyx/plan/11-color-through-send.md` (Intent).

**Issue:**
Card 11 calls `run.newSendContext()` and `r.newSendContext(state)`.
Card 3 introduces the `sendContext` builders (Intent: "its callers build one") but lists only `sendContext` among its created symbols, and card 11's `Uses` names `sendVerified` only.
A reader of the plan cannot find where the builders come from.

**Fix:**
Add `Run.newSendContext` and `Runner.newSendContext` to card 3's Create list and to card 11's `Uses`.

## Prior-round re-evaluation

Round 1's F2 (card 9 adds `PATTERN-shuttle-stop` while the orphaned removers survive to card 10) was disputed by the fixer.
I raised the same point independently before reading round 1 and withdrew it on re-evaluation: the removers have no callers at card 9's commit, so no code path removes a strand outside shuttle, and the spec's "last caller and the symbol are two cards" rule licenses card 10 as the final `Delete` card.
Round 1's F1 (the color decisions carried by no card) is carried by cards 11 and 12 and checked above; the cards follow cards 3 and 4, on which they depend, and cover start's color play and `Runner.TypeColor`.
Round 1's F3 hedges are resolved for cards 1, 5 (naming `TestStartup_SendFailuresTearDown`) and 8; the remaining choices are in F3 above.
