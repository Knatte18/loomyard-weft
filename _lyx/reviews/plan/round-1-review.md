---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-pause-sequence.md:102
    summary: Card 8 puts the new pure unit test TestShuttlePairReadings in internal/pausecli/cli_test.go, which is a //go:build integration file, so a tier-1 test that spawns nothing runs only in the integration tier, breaking PATTERN-test-speed's lowest-fitting-tier rule
    fix: In 08-pause-sequence.md add the Create bullet `internal/pausecli/deps_test.go` after the `plan:internal/pausecli#shuttlePair.InterruptSettle` bullet, and change the Intent bullet at line 102 from "New `TestShuttlePairReadings` in `cli_test.go`" to "New `TestShuttlePairReadings` in a new untagged file `internal/pausecli/deps_test.go`, whose file comment says it pins the readings `shuttlePair` derives from a session state; it spawns nothing, so it stays out of the integration-tagged `cli_test.go`". The overlay of this target change passes `lyx loom validate-plan --overlay` with no new blocking finding.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-inject-refusal-and-pattern.md:21
    summary: Inject's refusal compares only PaneInput.Key names against the engine's interrupt sequence, yet reed's SendText types Text with tmux send-keys -l, so a Text input holding the ESC byte, or a tmux alias key such as C-[, still delivers the interrupt key unguarded; the card's stated bound (Inject delivers every key but the interrupt key) and D7's bound (no Inject caller can carry one past the guard) do not hold, and the Intent names neither path
    fix: In card 5's Runner.Inject bullets, either close both paths (refuse any Text input that holds a control character, which is provider-neutral, and match tmux aliases of the interrupt key such as `C-[` for `Escape`), adding rows `{Text "\x1b"}` and `{Key "C-["}` to TestRunner_Inject_RefusesAnyInterruptKey, or, at minimum, rewrite the Bound bullet and the matching bullet of pattern/PATTERN-guarded-interrupt.md to state that the refusal matches key names only, name the ESC-in-Text and key-alias paths it lets through, and state what bounds them (Inject has no production caller, and creel reaches its Esc through the guarded Interrupt).
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/08-pause-sequence.md:86
    summary: Card 8's Intent changes the comment of the existing constant maxInterrupts to say guarded interrupts, but the card's Edit list does not name its member glyph
    fix: In 08-pause-sequence.md add `internal/pauseengine#maxInterrupts` to the **Edit:** list, after `internal/pauseengine#Pair`. The overlay passes `lyx loom validate-plan --overlay` with no new blocking finding.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/08-pause-sequence.md:72
    summary: Card 8 prescribes two new refused branches in interruptSession, a failed Background reading (background reading failed) at step 2 and a failed PaneRunningTurn reading (pane reading failed) at step 6, but fakePair has no way to fail either reading and no TestPause row reaches them
    fix: In card 8's fakePair bullets add `backgroundErr map[string]error` and `paneErr map[string]error`, answered by Background and PaneRunningTurn, and add one TestPause row, for example "a failed background or pane reading is refused with its prefix", that runs a session with busy [false] and a background error, and a session with busy [true] and paneRunning error after two keys, asserting each Refused reason starts with its planned prefix, background reading failed and pane reading failed.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/04-guarded-interrupt.md:75
    summary: The bound card 4 states for a failed picker capture in awaitIdleSession, that the idle wait's polls hold on an unreadable pane or one with no input box, holds only for a SessionCycler engine; for an engine without SessionCycler there are no polls, so a failed picker capture lets the send type into a possibly open picker, and no TestGuardedPickerClose row reaches the skip branch
    fix: In card 4's awaitIdleSession bullet, either fail closed in the branch without SessionCycler (a failed picker capture returns a wrapped capture error and types nothing), or state that bound as it is, that the branch types after a failed picker capture, bounded by requireReadyAgentPane having just read a ready pane and by every production engine implementing SessionCycler. Also add a TestGuardedPickerClose row for the runner-send op with `captureErrs [boom, nil]`, asserting zero SendKey calls and the idle wait's outcome, so the skip branch is reached.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/07-shuttle-cli-guard.md:19
    summary: Card 4's commit changes what lyx shuttle interrupt does (it now fails on a pane with no running turn and closes an open picker), but the verb's help moves only in card 7's later commit, against the record's constraint that the lyx shuttle interrupt help lands in the same commit as the behavior and CLAUDE.md's same-commit docs rule
    fix: Move `internal/shuttlecli#shuttleCLI.interruptCmd` from card 7's **Edit:** list to card 4's **Edit:** list, and move the help bullets (the Long text and the new Short) from card 7's Intent into card 4's Intent. Card 7 keeps its CLI test and the shuttlecli budget row. The overlay of both cards passes `lyx loom validate-plan --overlay` with no new blocking finding.
---

### [BLOCKING] TestShuttlePairReadings lands in an integration-tagged file

**Location:** _lyx/plan/08-pause-sequence.md:102

**Issue:** `internal/pausecli/cli_test.go` opens with `//go:build integration` and drives the verbs over a `hubforge` hub.
Card 8 puts `TestShuttlePairReadings` in it, a table over `SessionState` values that calls the pure `readTurn` and spawns nothing.
`PATTERN-test-speed` puts a test in the lowest tier it fits, and a test that needs nothing is tier 1, untagged.
As planned, the card gate's tier-1 run over `internal/pausecli` never runs the test, and `go test ./...` in `## verify:` does not either; only the integration tier does.
`internal/pausecli/testmain_test.go` is untagged, so an untagged `deps_test.go` runs under the package's existing `TestMain`.

### [MEDIUM] Inject's interrupt-key refusal is bypassable through Text and key aliases

**Location:** _lyx/plan/05-inject-refusal-and-pattern.md:21

**Issue:** The planned check refuses an input whose `Key` equals a `Key` of `r.engine.InterruptSequence()`.
`playInputsSent` sends a `Text` input through `reed.SendText`, and `reedengine`'s `SendText` runs `tmux send-keys -l` with the text as given (`internal/reedengine/io.go:106`, `sendKeysLiteralArg` only guards a leading `-`).
A `Text` of `"\x1b"` therefore reaches the pane as Escape.
tmux also accepts `C-[` as the Escape key, and its name differs from `"Escape"`.
The card's Bound bullet says Inject "delivers every key but the provider's interrupt key", and D7's bound says "no Inject caller can carry one past the guard"; both paths contradict that, and the card's Intent and the PATTERN background name neither.
Under the rubric's attack-surface rule, the card must either close the paths or state what they let through and what bounds them.

### [LOW] maxInterrupts is edited but not listed

**Location:** _lyx/plan/08-pause-sequence.md:86

**Issue:** The Intent says "update its comment to 'guarded interrupts'" for `maxInterrupts` in `internal/pauseengine/pause.go`.
The constant is a member with its own glyph, and the Edit list names `Pair`, `PauseReport`, `Pause` and `interruptSession`, but not it.
`TestPause_RacingResumeLeavesADecodableRecord` reads the constant, so it stays.

### [LOW] Two new pause refusal branches have no reaching test

**Location:** _lyx/plan/08-pause-sequence.md:72

**Issue:** Step 2 sends a failed `Background` reading to `Refused` with "background reading failed: ".
Step 6 sends a failed `PaneRunningTurn` reading to `Refused` with "pane reading failed: ".
The planned `fakePair` answers `Background` from a `background` map and `PaneRunningTurn` from a `paneRunning` map, so it has no way to fail either reading.
None of the listed `TestPause` rows reaches these branches, so a wrong prefix or a dropped branch passes the suite.

### [LOW] The failed-picker-capture bound in the send path is wrong for engines without SessionCycler

**Location:** _lyx/plan/04-guarded-interrupt.md:75

**Issue:** The card says that after a failed picker capture "the idle wait goes on as today; its polls read the pane again and hold on a pane they cannot read or that shows no input box, so nothing is typed into an unread picker".
In `awaitIdleSession` (`internal/shuttleengine/send.go:105`), an engine without `SessionCycler` runs only `requireReadyAgentPane` and the operator-activity hold, then returns nil, so `sendVerified` types at once.
For that branch the stated bound does not hold.
`shuttlefake.Engine` implements `RewindPickerReader` after card 3 and is not a `SessionCycler`, so the gap is reachable in the tree.
No row of `TestGuardedPickerClose` scripts a capture error on the picker check, so the skip branch has no reaching test either.

### [LOW] The lyx shuttle interrupt help trails its behavior change by three commits

**Location:** _lyx/plan/07-shuttle-cli-guard.md:19

**Issue:** Card 4's commit changes `Runner.Interrupt` so that `lyx shuttle interrupt` fails on a pane with no running turn and closes an open picker.
The verb's `Short` and `Long` help still describe an unconditional interrupt until card 7's commit.
The decision record's constraints list "the `lyx shuttle interrupt` help" among the docs that land in the same commit, and CLAUDE.md says observable CLI behavior and its docs land in the same commit.
Card 7's `RunE` is unchanged, so moving the help edit into card 4 costs that card one shuttlecli member and nothing else.

## Rejected

- Card 5's rewording of seven existing PATTERN lines is not scope beyond the record: `PATTERN.md` sits 20 bytes under `pattern.HeadroomBytes` (15 544 of 15 564), the rewording measures +16 bytes with the new entry (15 560), no rule changes, and each dropped detail is still in its background file (`PATTERN-lyx-temp-root.md` keeps the `""` case, `PATTERN-pane-text-injection.md` keeps the blind Enter).
- Card 9's watcher change is licensed by D4's watcher half, D3's Info log and D8's orch surface, and keeps `PATTERN-orch-pane-single-writer` and `PATTERN-daemon-wakeups`.
- Card order holds: cards 1 to 3 use only existing symbols and earlier cards' declarations, and every later `Uses` points at an earlier card.
- `## verify:` runs `./...` in the template's order, covering every targeted package, integration tier included.
- Card 4's claim that every engine implements `InterruptGuard` holds: the only `Engine` implementers are `claudeengine.Claude`, `shuttlefake.Engine` and the package's `fakeEngine`, and every other engine fake in `internal/shuttleengine` embeds `fakeEngine`.
- No production path besides `Run.Interrupt`, `Runner.Interrupt` and `Runner.Inject` plays the interrupt key, so the new PATTERN entry's chokepoint claim matches the tree, apart from F2.
- Card 2's picker reading (the subtitle needle plus the no-box condition) keeps within D8: the fixture classifies `StartupReady` under `Claude.Startup` and has no input box under `inputBoxInterior`, as the card's new rows pin.
- The re-sign arrow does not apply anywhere: no edited member's parameters, results or receiver change, and `SessionBusyError` is a new type on a `Create` label.
