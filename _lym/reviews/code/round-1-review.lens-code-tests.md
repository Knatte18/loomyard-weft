```yaml
lens: code-tests
outcome: reported
```

Task restated: code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Behaviour → pinning test**
- Wait start is the first read at the row (any presence), no reminder before the interval: `TestWaitReminders/NoneBeforeTheInterval`, `TerminalPresenceGetsNoneAndReturningRemindsAtOnce…` (the 46-minute line proves the start was recorded while terminal).
- One line per interval, text `<slug>: waiting at <row> for <n> minutes`: `OneLinePerIntervalWhileTheWaitLasts`.
- Sampler-Session reminded; a row outside `Rows` not: `SamplerSessionIsReminded`, `ARowOutsideTheListIsNot`.
- Rows list wired as PR-Review and Sampler-Session without Beam-Await: `TestHubDobbyDeps_WaitReminders` (`Rows`).
- Return to slack reminds at once, whatever the last reminder: `AReturnToSlackRemindsAtOnceWhateverTheLastReminder`.
- A pause dobby read restarts the wait and clears it: `APauseTheDobbyReadRestartsTheWait`, `TestWaitReminders_RecordTheWaitInTheStateRecord`.
- A pause and resume dobby did not read restarts the wait and its reminders: `APauseAndResumeNoCheckSawRestartsTheWaitAndItsReminders`.
- A resume before the wait began moves nothing: `AResumeBeforeTheWaitBeganMovesNothing`.
- Clears on leaving the row, on the child not running, and on the batten run not running: `LeavingTheRowEndsTheWait`, `AChildThatStopsRunningEndsTheWait`, `ARunBattenStoppedEndsTheWait`.
- A move to the other row starts a new wait: `MovingToTheOtherRowStartsANewWait`.
- A failed post is retried at the next check without advancing the last-reminder time: `APostThatFailsIsTriedAtTheNextCheck`.
- A restart keeps the start and may remind once early: `ARestartKeepsTheStartAndMayRemindOnceEarly`.
- Unreadable child, resume, presence or interval: `AChildThatDoesNotRead…`, `AResumeThatDoesNotRead…`, `APresenceThatDoesNotReadGetsNone`, `AnIntervalThatDoesNotReadGetsNone`.
- At most one check a minute: `TestWaitReminders_CheckAtMostOnceAMinute`.
- State record carries `OperatorWait` and clears it: `TestWaitReminders_RecordTheWaitInTheStateRecord`.
- `LatestResumed` per slug, hub scope, order-independent, none for a live entry or an empty record: `TestLatestResumed`.
- `LatestResume` reader returns zero with no record and the resume time with one: `TestHubDobbyDeps_WaitReminders`.
- `Away` reader, with presence slack and with no record: `TestHubDobbyDeps_WaitReminders`.
- `Interval` reader defaults to 30 minutes: `TestHubDobbyDeps_WaitReminders`.
- Wiring on both the prime's and a beam's dobby (`Post` non-nil): `TestHubDobbyDeps_WaitReminders`.
- Check runs on every tick that reaches `waitReminders` in `loop.go`: pinned indirectly by the harness `h.tick`.
- `Post` addressing and signing (to `<shortname>:SLACK`, signed `DOBBY` for the beam's slug): **none**.
- `remind_after_min` under 1 refused: not changed by this diff, and `LoadConfig` already enforced it. The engine's interval-read-failure row covers how the check reacts.
- Config and `template.yaml` wording: prose only, exempt.

**Findings**

1. MEDIUM, design, `internal/beamcli/dobbydeps.go:263-282` (`waitReminderDeps` `Post`), evidence `internal/beamcli/dobbydeps_integration_test.go:200-202`.
   - No test calls `Post`. The wiring test only checks it is non-nil.
   - A wrong program name or recipient, wrong scope slug, or a swallowed post error would pass every test, and that line is the one that reaches the operator.
   - Add a row, as `TestHubDobbyDeps_WaitReminders` does for the other seams, that posts a line and reads it from `<shortname>:SLACK`'s mailbox. The `landnotice_creel_test.go` pattern is a model.

2. LOW, design, `internal/dobbyengine/loop.go:611` (`d.waitReminders` placement) and the `doc.go` bound "while an anchor change stands no wait is checked".
   - No row shows that a standing anchor change suppresses the check.
   - Add a `loop` test row only if that gating is the intended behaviour.

3. LOW, design, `internal/dobbyengine/waitreminder.go:66` (`run.CancelMarker` skip) and `:49` (nil `Post` or nil `ReadChild` makes the check a no-op).
   - Neither branch has a row.
   - A `CancelMarker` row and a nil-seam row in `TestWaitReminders` would pin them.

4. NIT, design, `internal/dobbyengine/state.go:81`.
   - There is no test that a state record without `operator_wait`, or with an unknown key, still loads and keeps its keys.
   - Tolerance comes from `internal/state` and the generic state tests, so add this only if the project wants a per-field row.

**Test economy:** I found no duplicated test. `TestWaitReminders_RecordTheWaitInTheStateRecord` overlaps `APauseTheDobbyReadRestartsTheWait` slightly, but it pins the state record, which the table test does not read. The acceptance criterion's paused-and-resumed case is covered both with and without dobby reading the pause.

**Tool calls:** none made. The context already held every file this lens needed.
