---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/08-pause-sequence.md:105
    summary: TestShuttlePairReadings tests the unexported pure helper readTurn instead of the shuttlePair readings that use it, against PATTERN-test-economy, and the card's claim that a Busy regressing to Name == SessionBusy fails the test is false, since Busy is never called
    fix: In card 8's TestShuttlePairReadings bullet (lines 104-109), make the table drive shuttlePair.Busy and shuttlePair.Background instead of readTurn. Build the shuttlePair from a temp lyxcwd.Location (HubPath, WorktreeName) and a shuttleengine.Config whose RunDir is a temp directory, seed one run directory per row with a run.json whose outcome is running and an events file, and give the pair an engine whose session-signal reading yields the row's signals (a turn start for busy/turn; a turn end carrying the tasks {Label sleep 600} and {ID bsh2} for busy/background; a turn end with none for idle-done). Assert Busy's bool and Background's labels and bool per row. Keep readTurn as the private helper both methods call, reached only through them, and keep the sentence that a Busy reading Name == SessionBusy fails the busy/background row, which is then true. No target list changes.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-pause-sequence.md:80
    summary: interruptSession sleeps the settle once or twice per busy session inside the sequential per-session loop, so a pause over N sessions in a running turn takes at least N times 2 s, and up to N times 4 s, before it returns
    fix: In card 8's interruptSession steps (lines 72-88), restructure the interrupt step into passes over the covered sessions instead of one session at a time. Pass 1 runs steps 1-3 for every session (reading, claim, first guarded key). Then one deps.Sleep(settle). Pass 2 runs steps 4-5 for every session whose first key went (Busy, claim, guarded second key). Then one deps.Sleep(settle), and pass 3 runs step 6's PaneRunningTurn for every session whose second key went. Keep the claim before each key, so a resume during either settle stops the step with no further key, and keep every list outcome as the steps state it. Add to the busy-twice TestPause row a second busy session and assert the clock elapsed is exactly 2 settles (4 s), so the row fails if the settle goes back inside the per-session loop. No target list changes.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/05-inject-refusal-and-pattern.md:33
    summary: The decision record's acceptance criterion has every existing Inject refusal test that passes a lone Escape to reach another refusal switch to a non-interrupt input, but card 5 leaves the two told-path tests passing Escape and relies on check order instead
    fix: In 05-inject-refusal-and-pattern.md add `internal/shuttleengine#TestNewRunner_RefusesUnusableToldPaths` and `internal/shuttleengine#TestNewDetachedRunner_RefusesUnusableToldPaths` to the **Edit:** list after `internal/shuttleengine#TestRunner_Inject_UnknownGUID_RefusesBeforeTouchingReed`. Replace the Intent bullet at line 33 with one saying both tests' `Inject` calls pass a lone C-u key input instead of a lone Escape key input, so their told-path refusal does not depend on the told check running before the interrupt-key refusal. The overlay of this card passes `lyx loom validate-plan --overlay` with no new blocking finding.
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/plan/05-inject-refusal-and-pattern.md:49
    summary: Card 5 says the new entry is paid for by tightening seven existing lines, but the replacement block below it rewrites eight
    fix: In 05-inject-refusal-and-pattern.md line 49, replace "by tightening seven existing lines" with "by tightening the existing lines below", so the sentence carries no count.
---

### [BLOCKING] TestShuttlePairReadings pins a private helper, not the pair's readings

**Location:** _lyx/plan/08-pause-sequence.md:105

**Issue:** `pattern/PATTERN-test-economy.md` says a test targets a module's public surface and "an unexported helper is tested through the surface that uses it".
The planned `TestShuttlePairReadings` is a table over `SessionState` values that calls only `readTurn(state)`, an unexported pure function card 8 creates in `internal/pausecli`.
The card's stated reason for the test is that "a `Busy` that went back to reading `Name == SessionBusy` fails here", but the test never calls `Busy` or `Background`.
`shuttlePair.Busy` today reads `reading.State.Name == shuttleengine.SessionBusy` (`internal/pausecli/deps.go`), which is the D1 bug.
If the implementer leaves that line, or `Busy` stops calling `readTurn`, the test still passes.
`shuttlePair.Busy` and `Background` touch neither reed nor the runner: they read `shuttleengine.ReadSessionStates(p.cfg, p.location.AnchorPath(), p.engine, now)`, and `lyxcwd.Location` is a plain struct, so an untagged test can drive them over a temp run directory.

### [MEDIUM] The pause's settle runs once per busy session, in series

**Location:** _lyx/plan/08-pause-sequence.md:80

**Issue:** `Pause` calls `interruptSession` for each covered session in turn (`internal/pauseengine/pause.go`, the `for _, s := range covered` loop).
Card 8 puts `deps.Sleep(s.pair.InterruptSettle())` at step 4 for every session whose first key went, and again at step 6 for every session whose second key went.
With the 2-second settle, each session in a running turn adds at least 2 s and up to 4 s, and every later session's first Esc waits behind the earlier sessions' settles.
A hub with several drivers and webster workers mid-turn then takes tens of seconds to pause, while `docs/overview.md` describes the pause as making the hub safe to suspend within seconds.
D4 only requires each session's retry to wait the settle after that session's own first key, so one shared settle per pass meets it.
No TestPause row has two busy sessions and asserts elapsed time, so the per-session cost goes unseen.

### [LOW] Two told-path Inject tests keep passing a lone Escape

**Location:** _lyx/plan/05-inject-refusal-and-pattern.md:33

**Issue:** The decision record's acceptance criteria say "the existing Inject refusal tests that pass a lone `Escape` to reach another refusal switch to a non-interrupt input".
`TestNewRunner_RefusesUnusableToldPaths` (`internal/shuttleengine/run_test.go:83`) and `TestNewDetachedRunner_RefusesUnusableToldPaths` (`run_test.go:137`) both call `runner.Inject("strand-1", []PaneInput{{Key: "Escape"}})` to reach the told-path refusal.
Card 5 switches the dead-strand and unknown-GUID tests but keeps these two, on the grounds that the told check runs first.
That leaves the tests correct only because of the order of the checks inside `Inject`, which the criterion was written to avoid.

### [NIT] Card 5 miscounts the PATTERN lines it rewrites

**Location:** _lyx/plan/05-inject-refusal-and-pattern.md:49

**Issue:** The sentence says "tightening seven existing lines", and the block below it replaces eight: `orch-pane-single-writer`, `pane-text-injection`, `lyx-temp-root`, `friction-capture`, `mutation-record`, `plan-generation`, `gate-self-check-parity` and `parent-directive`.
The byte total holds with all eight: replacing the current eight lines (1768 bytes) with the planned ones (1594 bytes), plus the new entry (190 bytes), takes `PATTERN.md` from 15 544 to 15 560 bytes, under `pattern.HeadroomBytes` (15 564).

## Rejected

- Card 4's guarded interrupt and picker close match the tree: only `InterruptSequence` holds an `Escape` key in production code, the only `Engine` implementations are `claudeengine.Claude`, `shuttlefake.Engine` and `fakeEngine` (every other test engine embeds it), and `awaitIdleSession` is the one idle wait both `sendVerified` and `sendSequence` go through.
- Card 4's `TestGuardedPickerClose` capture scripts reach the branches they name: `pickerEngine` gets `SessionCycler` and `InputBoxReader` through `readinessEngine`, so the send ops take the `requireLiveStrand` branch with no ready-pane capture, and `pickerOnlyEngine` takes the `requireReadyAgentPane` branch.
- Card 2's picker fixture has no `─` rule line, so `inputBoxInterior` finds no box, and its caret makes `Claude.Startup` read `StartupReady`, as the new `TestInputBoxText` and `TestStartup_Classification` rows claim. The two needles match `pane-turn-running.txt` only, among the named fixtures.
- Card 7's CLI test works over `shuttlefake`: `Startup` defaults to `StartupReady`, the engine is no `SessionCycler`, and `Runner.Start` sends nothing when the launch carries no prompt line, so only the rows' own calls reach the picker capture.
- Card 9's restructure of `handleHubEntry` is licensed by D4's watcher half. It keeps the existing rows' outcomes: with `fakeSession`'s settle defaulting to 0, the re-probe on the tick after the grace end reads the orch idle in the notice row.
- The test-budget raises match the current counts (claudeengine 108, shuttleengine 168, shuttlecli 25, pausecli 3), and no card adds a top-level test to `internal/orchengine`, `internal/pauseengine` or `internal/testkit/shuttlefake`.
- Card order, verify coverage and member glyphs hold: every `Uses` points at an earlier card, `## verify:` runs `./...` across every tier the rubric requires, and no edited member's signature changes, so no re-sign arrow is missing.
