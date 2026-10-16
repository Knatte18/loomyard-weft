# Card 5 — shuttle-interrupt-restoring

**Create:**
- `plan:internal/shuttleengine#Runner.InterruptRestoring` -> `func (r *Runner) InterruptRestoring(guid string) error`
- `plan:internal/shuttleengine#ErrInterruptClearFailed` -> `var ErrInterruptClearFailed error`
- `plan:internal/shuttleengine#clearRestoredPrompt` -> `func clearRestoredPrompt(reed ReedOps, engine Engine, clock Clock, guid string) error`
- `plan:internal/shuttleengine#restoreReed` -> `type restoreReed struct`
- `plan:internal/shuttleengine#restoreReed.SendKey` -> `func (r *restoreReed) SendKey(guid, key string) error`
- `plan:internal/shuttleengine#restoreReed.CapturePane` -> `func (r *restoreReed) CapturePane(guid string) (string, error)`
- `plan:internal/shuttleengine#restoreReed.CapturePaneStyled` -> `func (r *restoreReed) CapturePaneStyled(guid string) (string, error)`
- `plan:internal/shuttleengine#restoreEngine.Startup` -> `func (e *restoreEngine) Startup(capture string) StartupState`
- `plan:internal/shuttleengine#styledRestoreEngine` -> `type styledRestoreEngine struct`
- `plan:internal/shuttleengine#styledRestoreEngine.PlainBoxCapture` -> `func (e *styledRestoreEngine) PlainBoxCapture(styled string) string`
- `plan:internal/shuttleengine#restoreEngine` -> `type restoreEngine struct`
- `plan:internal/shuttleengine#restoreEngine.InputBoxText` -> `func (e *restoreEngine) InputBoxText(capture string) (string, bool)`
- `plan:internal/shuttleengine#restoreEngine.SubmitSettle` -> `func (e *restoreEngine) SubmitSettle() time.Duration`
- `plan:internal/shuttleengine#restoreEngine.TypeSequence` -> `func (e *restoreEngine) TypeSequence(text string) []PaneInput`
- `plan:internal/shuttleengine#restoreEngine.RestoredPromptClearSequence` -> `func (e *restoreEngine) RestoredPromptClearSequence(capture string) []PaneInput`
- `plan:internal/shuttleengine#TestRunner_InterruptRestoring` -> `func TestRunner_InterruptRestoring(t *testing.T)`

**Edit:**
- `internal/shuttleengine/doc.go`
- `//pattern/PATTERN-guarded-interrupt.md`
- `//pattern/PATTERN-pane-text-injection.md`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#guardedInterrupt`
- `plan:internal/shuttleengine#boxCapture`
- `internal/shuttleengine#readBox`
- `plan:internal/shuttleengine#blankMarkedSpans`
- `internal/shuttleengine#liveStrand`
- `internal/shuttleengine#notShuttleOrLiveStrandError`
- `internal/shuttleengine#Runner.engineFor`
- `internal/shuttleengine#playInputs`
- `plan:internal/shuttleengine#RestoredPromptClearer`
- `internal/shuttleengine#InterruptGuard`
- `internal/shuttleengine#InputBoxReader`
- `internal/shuttleengine#fakeReed`
- `internal/shuttleengine#fakeEngine`
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#newFakeClock`
- `internal/shuttleengine#liveStrandStatus`
- `internal/shuttleengine#stubInputSleep`

**Intent:** Decision D3, shuttle half: an interrupt entry point for creel that clears the prompt Claude restores into the box after lyx's own interrupt key, a new entry point of `PATTERN-guarded-interrupt`.

`Runner.InterruptRestoring(guid)` resolves the strand's engine as `Runner.Interrupt` does: the run record's engine, and for a strand with no run record the live strand's provider through `liveStrand`, refusing an untracked guid with `notShuttleOrLiveStrandError`.
It reads the settle from that engine's `InterruptGuard.InterruptSettle`, never from `Runner.InterruptSettle`, which needs a run record.
In order:
1. One `boxCapture` and `readBox` of it is the before-reading; a failed capture fails the call and sends nothing.
2. The key goes only through `guardedInterrupt`, whose errors (`ErrNoRunningTurn`, `ErrRewindPickerOpen`, a failed capture, a pane not ready) return unchanged, with no clear.
3. `clearRestoredPrompt` runs only when the before-reading read a box and it was empty, and the engine has `RestoredPromptClearer` and `InputBoxReader`; otherwise the call returns nil as `Runner.Interrupt` would.
   It sleeps the settle on the runner's clock and reads the box; when that read finds the box empty it sleeps one more settle and reads once more, and a box still empty returns nil.
   A read whose box is unreadable sends nothing and returns nil.
   A box holding text gets the guard reading the stray-character clear takes, from the same capture plus one activity query: the provider pane reads ready (`Startup` is `StartupReady`), no running turn (`RunningTurn` is false), the box is readable, and `reed.OperatorActive` answers not active.
   A guard reading that reads the pane not ready, a running turn or an active operator sends nothing and returns nil: the restored text stays, and the receiver's next send holds on it as a draft, which creel shows (card 6).
   The activity query is asked directly, not through `operatorActivityHold`, which holds nothing on a query error: here a query error fails the guard reading closed, as step 4 says.
   A passing reading plays `RestoredPromptClearSequence` of that capture, sleeps the engine's `SubmitSettle` and reads the box once more; text still there is logged at Info and nothing more is sent.
4. A failure after the key (a capture, the guard reading's activity query, a play of the clear) is logged at Warn and returned as an error wrapping `ErrInterruptClearFailed`, which says the interrupt key was sent and the clear did not finish, so a caller treats the interrupt as sent; a failed guard reading sends no clear key.

Bound, stated in the doc comment: the clear fires only on text that appeared in a box read empty before lyx's own key, at most once per call and never without a passing guard reading; its keys are the provider's restored-prompt clear, no Escape, `C-c` or Enter, at most two per line the read showed; that text is the stopped turn's own prompt, already in the transcript.
`Runner.Interrupt`, `Run.Interrupt` and their callers (`lyx shuttle interrupt`, tender's pause, the orch watcher) are unchanged.

The fakes: `restoreReed` embeds `*fakeReed` and its fields script the pane: `before`, the box text before the key (empty for an empty box); `idleBefore`, which draws no `RUNNING` before the key; `suggestion`, a placeholder drawn in an empty box before the key, plain in `CapturePane` and wrapped in `~` marks in `CapturePaneStyled`; `restored`, the lines drawn after the key; `lateRestore`, which draws them only from the second capture after the key; and `afterHead`, the first line of every frame after the key, `IDLE` when empty.
Before the key it draws `RUNNING` (unless `idleBefore`) and `❯ ` plus `before` or the suggestion; after an `Escape` it draws `afterHead` and a box of the restored lines (`❯ <line1>` then `  <line>` rows), or no caret line at all when `afterHead` is `NOBOX`.
`restoreReed.CapturePane` and `restoreReed.CapturePaneStyled` each call the embedded `fakeReed`'s capture first, so its `CaptureErrQueue` fails a scripted capture, counted over every capture of the call.
`restoreReed.SendKey` deletes the caret row's text on `C-u`, and on `BSpace` at an empty last row removes that row so the caret joins the row above, else deletes one rune, as Claude did live; it records every key through the embedded `fakeReed`, whose `SendKeyErrQueue` fails a scripted key.
`restoreEngine` is a `fakeEngine` (`RUNNING` read as a running turn, settle 2 s) with `Startup` answering ready unless the capture holds `SHELL`, `InputBoxText` reading the rows after the caret joined by a space, a zero `SubmitSettle`, a `TypeSequence` of the text, and `RestoredPromptClearSequence` built as Claude's is, sized from the first to the last row holding text with blank rows between them counted, or as one bare `C-u` when its `singleClear` field is set.
`styledRestoreEngine` embeds `*restoreEngine` and answers `PlainBoxCapture` with `blankMarkedSpans`, so the box readings of the call go through the styled capture.

`TestRunner_InterruptRestoring` is a new table test, since `TestRun_Interrupt_PlaysEscape` pins the unchanged guarded key and a restored box needs the stateful fake; its rows assert the keys sent, the final box and the error:
a two-row and a three-row restored prompt are emptied by `Escape` then `C-u BSpace C-u` and `C-u BSpace C-u BSpace C-u`;
a three-row restored prompt whose middle row is blank is emptied by `Escape` then `C-u BSpace C-u BSpace C-u`;
the `singleClear` engine's one `C-u` leaves text, returns nil and sends nothing after it;
a box that held a draft before the key gets the `Escape` alone;
a box showing only the suggestion before the key, on `styledRestoreEngine`, reads empty, so its restored prompt is cleared, while the same pane on `restoreEngine` reads a draft before the key and gets the `Escape` alone;
no restored prompt after the key returns nil with the `Escape` alone, the clock showing two settles slept;
a prompt drawn only at the second post-settle read is cleared, and the clock shows two settles slept before the clear;
a guard reading over `SHELL`, over `RUNNING` and with an active operator each get the `Escape` alone, return nil and keep the prompt;
a post-key box that is unreadable (`NOBOX`) gets the `Escape` alone and returns nil;
an engine without the restored-prompt clear (a plain `fakeEngine`) gets the `Escape` alone and returns nil;
a capture that fails before the key fails the call with no key sent;
a capture that fails after the key, an activity query that fails at the guard reading (`ActivityErrQueue`) and a failed `C-u` of the clear (`SendKeyErrQueue`) each return an error wrapping `ErrInterruptClearFailed` with the `Escape` sent, and the failed activity query sends no clear key;
a pane with no running turn (`idleBefore`) returns `ErrNoRunningTurn` and sends nothing;
a strand with no run record, resolved through the live strand's provider, has its restored prompt cleared;
and a guid with no run record that reed reports no live strand for (a `newFixture` without `withStrand`, the `fakeReed`'s `StatusQueue` from `liveStrandStatus(false)`) fails with `notShuttleOrLiveStrandError`'s error naming the guid and sends no key.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/shuttleengine` row rises from 178 to 179.

Docs in the same commit: `internal/shuttleengine/doc.go`'s interrupt paragraph gains `Runner.InterruptRestoring`;
`pattern/PATTERN-guarded-interrupt.md` names it as an entry point whose key goes only through the guarded interrupt and whose clear is the engine's box clear, never an interrupt key, and adds `TestRunner_InterruptRestoring` to the tests that pin it;
`pattern/PATTERN-pane-text-injection.md` adds the restored-prompt clear as a second bounded clear beside the stray-character one, with the bound above, and lists it among the explicit key deliveries.

**ImpactSummary:** Adds one Runner entry point nothing calls until card 8, plus docs; Runner.Interrupt and the three callers outside creel are untouched.
