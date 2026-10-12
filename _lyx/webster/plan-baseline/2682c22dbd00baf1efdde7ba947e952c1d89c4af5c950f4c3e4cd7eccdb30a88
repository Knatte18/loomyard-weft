# Card 8 — the pause interrupts only a running turn, settles before a guarded retry, and reports plainly

**Create:**
- `plan:internal/pauseengine#BackgroundSession` -> `type BackgroundSession struct`
- `plan:internal/pauseengine#fakePair.Background` -> `func (p *fakePair) Background(guid string) ([]string, bool, error)`
- `plan:internal/pauseengine#fakePair.PaneRunningTurn` -> `func (p *fakePair) PaneRunningTurn(guid string) (bool, error)`
- `plan:internal/pauseengine#fakePair.InterruptSettle` -> `func (p *fakePair) InterruptSettle() time.Duration`
- `plan:internal/pausecli#shuttlePair.state` -> `func (p shuttlePair) state(guid string) (shuttleengine.SessionState, error)`
- `plan:internal/pausecli#readTurn` -> `func readTurn(state shuttleengine.SessionState) (running bool, labels []string, background bool)`
- `plan:internal/pausecli#TestShuttlePairReadings` -> `func TestShuttlePairReadings(t *testing.T)`
- `plan:internal/pausecli#signalEngine` -> `type signalEngine struct`
- `plan:internal/pausecli#signalEngine.ParseSessionSignals` -> `func (e *signalEngine) ParseSessionSignals(data []byte) ([]shuttleengine.SessionSignal, int)`
- `plan:internal/pausecli#shuttlePair.Background` -> `func (p shuttlePair) Background(guid string) ([]string, bool, error)`
- `plan:internal/pausecli#shuttlePair.PaneRunningTurn` -> `func (p shuttlePair) PaneRunningTurn(guid string) (bool, error)`
- `plan:internal/pausecli#shuttlePair.InterruptSettle` -> `func (p shuttlePair) InterruptSettle() time.Duration`
- `internal/pausecli/deps_test.go`

**Edit:**
- `internal/pauseengine#Pair`
- `internal/pauseengine#maxInterrupts`
- `internal/pauseengine#PauseReport`
- `internal/pauseengine#Pause`
- `internal/pauseengine#interruptSession` -> `func interruptSession(deps Deps, entry pauserecord.Entry, sessions []session, report *PauseReport) (stop bool)`
- `internal/pauseengine#sendResumeLine`
- `internal/pauseengine#fakePair`
- `internal/pauseengine#newPair`
- `internal/pauseengine#fakePair.Interrupt`
- `internal/pauseengine#TestPause`
- `internal/pauseengine#TestResume`
- `internal/pauseengine/doc.go`
- `internal/pausecli#shuttlePair.Busy`
- `internal/pausecli#Command`
- `internal/pausecli#ResumeCommand`
- `internal/pausecli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#SessionState.RunningTurn`
- `internal/shuttleengine#ErrNoRunningTurn`
- `internal/shuttleengine#ErrRewindPickerOpen`
- `plan:internal/shuttleengine#SessionBusyError`
- `internal/shuttleengine#Runner.PaneRunningTurn`
- `internal/shuttleengine#Runner.InterruptSettle`

**Intent:**
D1, D2, D4 (pause engine), D5, D6 and D8's pause and resume surfaces.
The pause entry's shape does not change (`PATTERN-tolerant-records`); the new lists live in the report only.

`Pair` in `report.go`; the engine still imports no provider package (`PATTERN-shuttle-provider-seam`):

- `Busy` keeps its signature, and its contract becomes "the session is in a running turn": busy with any cause but background work.
- New `Background(guid string) (labels []string, background bool, err error)`: whether the session's turn ended with background work still running, with the outstanding tasks' labels.
- New `PaneRunningTurn(guid string) (bool, error)`: the guard's own pane reading.
- New `InterruptSettle() time.Duration`: the provider's settle time.
- The `Interrupt` comment says it goes through shuttle's guarded interrupt, which returns `ErrNoRunningTurn` when the pane shows no running turn and `ErrRewindPickerOpen` when it closed an open picker.

`PauseReport`:

- New `NoRunningTurn []string` (`json:"no_running_turn"`) and `Background []BackgroundSession` (`json:"background"`).
- `BackgroundSession` has `Strand`, `Slug` and `Tasks []string` (`json:"strand"`, `"slug"`, `"tasks"`).
- `Pause` initialises both to empty slices beside the others.
- The field comments say plainly what each list holds and whether resume types into it, per D5:
  - `Interrupted`: stopped after a key: the state read no running turn after the first key's settle, the guarded second key found none, or the pane read none after the second key's settle.
  - `NoRunningTurn`: the state read a running turn but the guarded first key found none on the pane; recorded, so resume gives each a line.
  - `AlreadyIdle`: no running turn and no background work; not recorded, no line.
  - `Background`: the turn ended with background work outstanding; not recorded, no line, nothing sent.
  - `StillBusy`: the pane still showed a running turn after the second key's settle.
  - `Refused` now also holds a failed guard capture and an open rewind picker, with the error's message.

The interrupt step runs in passes over the covered sessions instead of one session at a time, so a pause waits each settle once whatever the number of busy sessions.
`Pause`'s per-session loop keeps the caller's own session in `Skipped` and hands the rest to `interruptSession`, which now takes them all: `interruptSession(deps, entry, sessions, report) (stop bool)`.
The entry-live check and the claim stay before each key, so no key follows a resume, and an `errEntryGone` at any claim stops the whole step, as today.
Each settle is one `deps.Sleep` of the longest `InterruptSettle()` among the sessions the pass waits for, skipped when there are none.

Pass 1, for every session:

1. `Busy` (running turn).
   A read error goes to `Refused` as today.
2. Not running: `Background`.
   A read error goes to `Refused` ("background reading failed: ").
   A background session goes to `Background` with its labels, with no claim and no key.
   Otherwise the session goes to `AlreadyIdle`.
3. Running: claim (an `errEntryGone` stops the step, `errOwnedElsewhere` skips as today), then `Interrupt`.
   - `ErrNoRunningTurn` goes to `NoRunningTurn`, and the session stays recorded.
   - Any other error goes to `Refused`: the `ErrPaneNotReady` prefix is kept, and every other error, a picker or a failed capture included, uses its message as the reason.

Then one settle for the sessions whose first key went.

Pass 2, for every session whose first key went:

4. `Busy`.
   A read error goes to `Refused`; no running turn goes to `Interrupted`.
5. Still running: claim again, which is the entry-live check for the second key, so a resume during the settle ends the step with no second key; then `Interrupt`.
   - `ErrNoRunningTurn` goes to `Interrupted`.
   - Any other error goes to `Refused` as in step 3.

Then one settle for the sessions whose second key went.

Pass 3, for every session whose second key went:

6. `PaneRunningTurn`.
   A read error goes to `Refused` ("pane reading failed: "); a running turn goes to `StillBusy`, none to `Interrupted`.

At most `maxInterrupts` (2) `Interrupt` calls per session per pause, as today; update its comment to "guarded interrupts".
The grace's `anyBusy` reads `Busy`, so a background session no longer holds the grace (D1).

`sendResumeLine` (D1, D6): drop the `Busy` pre-check, so a recorded session gets its line attempt through the guarded send whatever its raw state reads.
Map the send error:

- `errors.As` to `*shuttleengine.SessionBusyError` returns its `Reason`, with no pane tail.
- Any other error returns `"the send failed: " + err.Error()`, which carries `ErrRewindPickerOpen`'s message for a picker.

No line is typed in either case.

`pausecli`'s `shuttlePair` (`deps.go`):

- A private `state(guid)` reads `shuttleengine.ReadSessionStates` once and returns the strand's `SessionState`, or the zero state when the strand has no running run.
- A pure `readTurn(state)` derives both readings: `running` is `state.RunningTurn()`, and `background` is true for `SessionBusy` with cause `SessionCauseBackground`, with one label per `Outstanding` task, its `Label` or its `ID` when the label is empty.
- `Busy` returns `readTurn`'s `running`, and `Background` its labels and `background`.
  `readTurn` stays the private helper both methods call, and is reached in tests only through them.
- New `TestShuttlePairReadings` in a new untagged file `internal/pausecli/deps_test.go`, whose file comment says it pins the readings `shuttlePair` derives from a run's session state; it spawns nothing and touches neither reed nor the runner, so it stays out of the integration-tagged `cli_test.go` and runs under the package's untagged `TestMain`.
  It drives `shuttlePair.Busy` and `shuttlePair.Background`, so a `Busy` that went back to reading `Name == SessionBusy` fails the busy/background row.
  - The file declares `signalEngine`, which embeds `*shuttlefake.Engine` and implements `shuttleengine.SessionSignalParser`: `ParseSessionSignals` returns its `signals` field whatever the data, with `consumed` the data's length.
  - Each row builds a `shuttlePair` from a `lyxcwd.Location` with `HubPath` a temp directory and `WorktreeName` "p1", a `shuttleengine.Config` whose `RunDir` is a temp directory, and a `signalEngine` carrying the row's signals.
    It seeds one run directory under `RunDir` holding a `run.json` whose `outcome` is "running", whose `strandGuid` is "g-a" and whose `eventsPath` names an events file it also writes.
  - It is one table asserting `Busy("g-a")`'s bool and `Background("g-a")`'s labels and bool per row, each with a nil error:
    - busy/turn, a turn start: running, not background;
    - busy/background, a turn end carrying the tasks `{Label: "sleep 600"}` and `{ID: "bsh2"}`: not running, background with labels `["sleep 600", "bsh2"]`;
    - turn ended, a turn end with no tasks: neither.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/pausecli` from 3 to 4.
- `PaneRunningTurn` delegates to `p.runner.PaneRunningTurn`, and `InterruptSettle` to `p.runner.InterruptSettle`.

Help and docs, in the same commit:

- `pausecli`'s `Command` `Short` becomes "pause the hub or one pair: hold agent and network work and interrupt running turns", and its `Long` says it interrupts every covered session still in a running turn with Esc, sends a second Esc only after the provider's settle time and only into a pane still showing a running turn, and reports sessions whose turn ended with background work under `background`.
- `ResumeCommand`'s `Long` replaces "reads busy or holds a draft in its input box" with "or whose guarded send is refused, as for a running turn, a draft, a dialog on screen or an open rewind picker".
- `pauseengine/doc.go` rewrites the Pause section's interrupt paragraph and the Resume section's no-line sentence to match: running turn only, the settle and the guarded retry, the lists, no busy pre-check and the guard's own reason.
- `docs/overview.md`'s pause bullet: "interrupts every covered session still in a running turn with Esc".

Tests, `pause_test.go`:

- `fakePair` gains:
  - `background map[string][]string`: a key present reads as background with those labels.
  - `backgroundErr map[string]error`: a key present fails `Background` with that error.
  - `paneRunning map[string]bool`, answered by `PaneRunningTurn`, which also counts its calls in `paneReads int` and runs an `onPaneRead func()` hook when set.
  - `paneErr map[string]error`: a key present fails `PaneRunningTurn` with that error, after the count and the hook.
  - `settle time.Duration`, answered by `InterruptSettle`.
  - `interruptErr map[string]error` becomes `interruptErrs map[string][]error`, consumed in order with the last repeating, like `busy`, an empty list answering nil.
  `newPair` initialises the new maps and sets `settle` to 2 s, so every pause waits a settle the clock can show.
- `TestPause` rows:
  - "the grace waits for a busy session and never past the grace end": set `p.settle = 0`, since the session stays busy through both keys and the row pins the clock at exactly the 10-second grace.
  - "the entry is on disk and the session recorded in it before the first Esc": add `len(p.interrupts) == 1` beside `Interrupted == [a]`.
    This is the "first Esc frees the pane" criterion: one key, then the state reads idle after the settle.
  - "a busy session is interrupted at most twice and reported still busy" becomes "busy sessions are interrupted at most twice, share each settle and are reported still busy", over a pair with sessions `a` and `b`, both busy `[true]` and `paneRunning` true.
    Record each `Interrupt`'s clock time through `onInterrupt` and the pane reads' through `onPaneRead`.
    Assert the interrupts `[g-a, g-b, g-a, g-b]`, each session's second key at least `p.settle` after its first, each pane read at least `p.settle` after the second keys, and `StillBusy == [a, b]`.
    Assert the clock elapsed is exactly 2 settles (4 s), so the row fails if the settle goes back inside a per-session loop, and the second gap fails if the settle after the second key is dropped.
  - New "a second Esc whose settle leaves no running turn on the pane is interrupted": busy `[true]`, `paneRunning` false; two interrupts, `Interrupted == [a]`, `StillBusy` empty.
  - New "a guarded second Esc that finds no running turn ends interrupted": busy `[true]`, `interruptErrs [nil, wrapped ErrNoRunningTurn]`; two `Interrupt` calls and `Interrupted == [a]`.
    Also assert `p.paneReads == 0`.
  - New "an open rewind picker found by the guarded second Esc is refused": busy `[true]`, `interruptErrs [nil, wrapped ErrRewindPickerOpen]`; two `Interrupt` calls and `Refused[0].Reason` contains "rewind picker was open".
  - New "a running state whose first guarded Esc finds no running turn is no_running_turn and stays recorded": busy `[true]`, `interruptErrs [wrapped ErrNoRunningTurn]`.
    One `Interrupt` call, `NoRunningTurn == [a]`, the live entry's sessions `[recorded("p1", "a")]`, `Interrupted` and `AlreadyIdle` empty.
  - New "a background session holds no grace, gets no Esc, is not recorded and is listed with its tasks": busy `[false]`, `background["g-a"] = ["sleep 600", "lyx shed step"]`, grace 10 s.
    The clock elapsed is 0, there are no interrupts, the entry has no sessions, and `Background == [{a, p1, [sleep 600, lyx shed step]}]`.
  - New "an open rewind picker found by the guarded Esc is refused with its message": busy `[true]`, `interruptErrs [fmt.Errorf("%w: strand g-a: one interrupt key closed it", shuttleengine.ErrRewindPickerOpen)]`.
    One interrupt, `Refused[0].Reason` contains "rewind picker was open".
  - New "a failed guard capture is refused": busy `[true]`, `interruptErrs [errors.New("shuttle: capture strand \"g-a\"'s pane before the interrupt: boom")]`; `Refused[0].Reason` contains "boom".
  - New "a resume during the settle sends no second Esc": busy `[true]`; `onInterrupt` installs a `clock.onSleep` that resumes the hub entry.
    One interrupt and `ResumedDuringGrace` true.
  - New "a failed background or pane reading is refused with its prefix": session `a` with busy `[false]` and `backgroundErr["g-a"] = boom`, and session `b` with busy `[true]` and `paneErr["g-b"] = boom`.
    `a` gets no interrupt and its `Refused` reason starts "background reading failed: "; `b` gets two interrupts and its `Refused` reason starts "pane reading failed: ".
  - "a pane refusal is reported as refused" uses `interruptErrs`.
- `TestResume`:
  - The row "a dead, busy or draft-holding session gets no line and each is named with its reason" becomes "a dead strand, a held send and an open picker get no line, each named with its reason".
    - `held` has send error `&shuttleengine.SessionBusyError{Reason: "a turn is running", Err: fmt.Errorf("%w: strand: a turn is running; retry once the session is idle. The pane's last lines:\ntail", shuttleengine.ErrSessionBusy)}`, and its reason is exactly "a turn is running".
    - `draft` has a `SessionBusyError` with reason `the input box holds a draft: "x"`, and its reason is exactly that.
    - `picker` has send error `fmt.Errorf("%w: strand g-picker: one interrupt key closed it", shuttleengine.ErrRewindPickerOpen)`, and its reason starts "the send failed: " and contains "rewind picker was open".
    - `dead` keeps "the strand is not live".
    - Nothing is sent to any of them.
  - New row "a recorded session that reads busy at resume still gets its line": `busy["g-a"] = [true]` and no send error.
    `Lines == [a]` and one send, so the busy pre-check is gone.
    It stands for a `no_running_turn` session whose raw state still reads busy/turn and for an entry recorded before this change whose session now reads busy/background, since `Busy` is no longer read at resume.
- `pausecli`'s `TestPauseAndResume_WriteLiftAndRefuseOverTheHub`: the empty-list loop also covers `no_running_turn` and `background`, and for those two asserts the key is present as a JSON array.

The record's by-hand criterion has two checks, against a live hub after landing, since no automated tier drives a real Claude pane through a pause:
- `lyx pause` against a driver idle at its prompt with a background `lyx shed step` sends no Esc and opens no picker.
- `lyx pause` against a driver mid-turn sends one Esc and leaves no picker.

They stay the operator's check, named here so they are not lost.

**ImpactSummary:** `Pair` gains three methods with two implementers (`fakePair`, `shuttlePair`); the pause and resume reports gain lists and change reasons, read only by `pausecli`'s envelope; no record shape changes.

**Commit:** 8: Pause interrupts only a running turn, settles before a guarded retry and lists background work
