---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/websterengine/verifyfixer.go:127
    summary: "Webster's awaitFixerStrand builds gatefixer.Deps without ShuttleCfg or AnchorRoot, so the new goneStrandReason calls shuttleengine.FindRun with an empty anchor and looks for run records under the relative path .lym/shuttle, resolved against the process cwd; a fix-plan or fix-verify fixer's dead reason now depends on the cwd, which PATTERN-cwd-resolution forbids (engines get absolute paths)."
    fix: "In awaitFixerStrand (internal/websterengine/verifyfixer.go), pass the shuttle config and the absolute anchor root into gatefixer.Deps, the same ShuttleCfg and AnchorRoot values webster already hands gatefixer.SpawnOrAttach when it spawns these fixers, threading them through awaitFixerStrand's parameters from FixVerifyAwait and FixPlanAwait. Add a TestFixVerifyAwait row whose fixer strand is gone and whose shuttle run record is marked interrupted, asserting DeadReasonInterrupted."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/gatefixer/rowgate.go:159
    summary: "In the board's own scenario, a driver restarted onto a left-live fixer record, the interrupted fixer is never replaced. The new wrap did not spawn that strand, so expired() reports true, the shuttle poll loop's probeExpiry finalizes (shuttleengine/wait.go:416-420), and evaluateGate(true) runs the entry's Final closure, which waits with replace=false and marks the record terminal as interrupted. An earlier non-fixer entry failing over the record does the same. The LeftLiveByARestartedDriver row calls entry.Gate() directly and never takes this path, and no row drives Final or a waitOnly entry over an interrupted fixer."
    fix: "In waitOnly (internal/gatefixer/rowgate.go), pass afterFixer as waitThenRun's replace argument, so the fixer entry's Final closure replaces an interrupted fixer like the fixer entry's Gate does. For the other entries (afterFixer false), leave a record whose wait ended interrupted live instead of marking it terminal in waitThenRun, so the fixer entry or its Final meets and replaces it. Update WrapRowGate's doc comment and the summary's sentence that only the carrying entry replaces. Add TestWrapRowGate rows that call entry.Final() and another entry's Gate over a left-live record whose fixer ends interrupted, asserting the replacement's spec on fixer 1's files for Final and a still-live record for the other entry."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/bouncer.go:2226
    summary: "A judge whose start failed with an error other than the not-ready kind still degrades to Stuck and spends bounce budget. RunGated returns such start errors unchanged (shuttleengine/run.go:1263-1269), among them a spawn the host refused for EAGAIN, which shedtransient.Class would class as transient; degrade swallows the error before the engine can class it. The judge never gave a verdict, which is the case the board entry says must not spend the budget."
    fix: "In judgeCall (internal/shedadapters/bouncer.go), when runErr is non-nil and no verdict was harvested, return the cancellation error if ctx is cancelled, and otherwise return an error wrapping runErr (with the producer and engine label, as judgeDiedError does) instead of calling degrade. Update judgeCall's doc comment. Add a TestBouncer_JudgeCall_ADiedJudgeIsAnInfrastructureFailure row whose fake shuttle returns a Run error, asserting no outcome, an empty pointer and errors.Is on that error."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: handler
    location: "_lym/landing/summary.md#Merge danger"
    summary: "The blast radius names only shed, but the diff changes boxCapture, which every shuttle send, idle probe, stray clear and interrupt read goes through (send.go:263 and :946, session.go:356, run.go:1387, interruptguard.go:77), the Claude engine's input-box reading tender revive relies on, the reed watchdog's OOM notices and the gate fixer await webster's fix-plan and fix-verify share."
    fix: "Rewrite the Blast radius line in _lym/landing/summary.md to name shuttle (every send-readiness and idle reading through boxCapture), the Claude engine's input-box reader and tender revive, reed's pane OOM notices, and webster's fix-plan and fix-verify fixer waits, beside shed."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatefixer/rowgate.go:49
    summary: "The interrupted note tells the writer that no replacement ran, but waitReplacingInterrupted returns the same interrupted outcome once the bound is passed, after one or more replacements ran; AnInterruptionPastTheCapStandsWithItsNote asserts two spawns beside this note."
    fix: "In fixerOutcome.note (internal/gatefixer/rowgate.go), drop the clause saying no replacement ran, so the text reads that the cold fixer was interrupted, by a machine stop or a killed pane, before it replied, and that any change it left uncommitted is in the worktree; or carry a replaced count in fixerOutcome and say how many replacements were interrupted too."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatefixer/rowgate.go:287
    summary: "A replacement whose SpawnOrAttach fails returns the interrupted outcome with a nil error and drops the spawn error: nothing logs it, the writer's findings omit its cause, and the transient mark SpawnOrAttach adds is lost. The first spawn, by contrast, puts its error in the findings (rowgate.go:215-218). No test row fails a replacement spawn."
    fix: "In waitReplacingInterrupted (internal/gatefixer/rowgate.go), keep the spawn error on the returned fixerOutcome in a new field, and have fixerOutcome.note say the replacement could not be started and give that error, as spawnAndJudgeAgain's line does. Add a TestWrapRowGate row with interruptedFirst 1 and a spawn error on the second spawn, asserting the failure's findings name that error."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/gatefixer/await.go:118
    summary: "goneStrandReason depends on the interrupted fixer's run directory surviving until Await reads it, but the reconcile stop removes the strand from reed state and any later shuttle Start in the worktree runs the orphan sweep, which deletes a run directory older than twice the startup timeout whose guid is not in reed state (shuttleengine/rundir.go:413-414). A fixer whose wait comes that late reads died, spends budget and is not replaced; the timing depends on when the stop last wrote run.json."
    fix: "In sweepOrphans (internal/shuttleengine/rundir.go), keep a run directory whose record resumableAfterInterruption accepts, so an interrupted run's record stays readable for gatefixer and for crash resume, and add a TestSweepOrphans row with an old stopped, interrupted record whose strand is gone, asserting the directory is kept."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatefixer/rowgate_test.go:556
    summary: "The markSpawned call for a replacement (rowgate.go:291-292) survives both mutants: no row checks Expired() while a replacement is waited on, and a wrong true there finalizes the writer's gate early through probeExpiry."
    fix: "In the SpawnedByThisWrap row of AnInterruptedFixerIsReplacedOnItsOwnFiles (internal/gatefixer/rowgate_test.go), set fakeFixer.onAwait to record entry.Expired() once the replacement is the live record, and assert it reported false."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/readiness.go:94
    summary: "While the styled and plain captures keep disagreeing, the case the fallback targets, boxCapture logs its Warn with a twelve-line tail on every call: each poll tick of the send-ready wait, each idle probe and each interrupt read."
    fix: "In boxCapture (internal/shuttleengine/readiness.go), log the disagreement at Warn once per strand GUID, keeping the GUIDs already warned in a package-level set guarded by a mutex, and at Debug after that."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/verifyfixer.go:246
    summary: "FixVerifyAwait's doc comment still states the old order, asking then timeout then a gone strand as died, though the gatefixer.Await it delegates to now reads the strand before the timeout and can return the interrupted reason."
    fix: "Rewrite the order sentence of FixVerifyAwait's doc comment (internal/websterengine/verifyfixer.go) to: dead when its newest turn ended without one (DeadReasonAsking), when its strand is gone (DeadReasonInterrupted for a run lym recorded as interrupted, DeadReasonDied otherwise), or when recovery_timeout_min has passed since the spawn (DeadReasonTimeout)."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatefixer/await_test.go:115
    summary: "Two branches the diff adds have no row: goneStrandReason returning died when the gone strand's run record is missing, and boxCapture keeping the styled reading when the plain capture fails."
    fix: "Add a TestAwait row (internal/gatefixer/await_test.go) that removes rec's run.json before goneStrand(fx), wanting DeadReasonDied; add a TestRunner_SessionIdle_ReturnsScriptedClassification row (internal/shuttleengine/session_test.go) with styledLosesBox and a failing plain capture, wanting the no-input-box hold."
  - id: F12
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/session_test.go:968
    summary: "No row wraps the API-error prompt over two box lines, though holdsOnlyAPIErrorPrompt's doc claims it matches, and no row covers judgeDiedError's not-started branch with an empty StartError."
    fix: "Add a TestInputBoxText row whose box holds the prompt wrapped over two lines (the caret line ending after 'Resend your message' and the next line holding 'to try again'), wanting empty text and ok true; add a TestBouncer_JudgeCall_ADiedJudgeIsAnInfrastructureFailure row with NotStarted true and an empty StartError, wanting ErrNotStarted."
---

### [BLOCKING] Webster's fixer wait resolves run records against the process cwd

**Location:** `internal/websterengine/verifyfixer.go:127`

**Issue:** `goneStrandReason` reads the gone fixer's run record through `shuttleengine.FindRun(deps.ShuttleCfg, deps.AnchorRoot, guid)`.
Webster's `awaitFixerStrand` builds `gatefixer.Deps{Stopper, Reed, Engines, Clock}` with neither field.
`runDirRoot` then joins an empty anchor with `.lym/shuttle`, a relative path resolved against whatever directory the process runs in.
The reason a fix-plan or fix-verify fixer ends with therefore depends on the cwd.
That breaks PATTERN-cwd-resolution, under which engines get absolute paths and only `lymcwd` resolves a cwd.
The diff changes the shared `Await` without wiring the new dependency into its second caller.

### [MEDIUM] A restarted driver never replaces the interrupted fixer

**Location:** `internal/gatefixer/rowgate.go:159`

**Issue:** The incident the board entry describes is a driver restarted onto a fixer record a reboot left live.
That wrap never spawned the record's strand, so `expired()` reports true.
While no boundary has been seen since the attach, the shuttle poll loop runs `probeExpiry` and finalizes (`internal/shuttleengine/wait.go:410-420`).
`evaluateGate(true)` then runs the fixer entry's `Final` closure (`wait.go:1887-1888`), which is `waitOnly(..., true, true)` and waits with `replace=false`.
The interrupted record turns terminal, and the writer gets the interrupted note.
The board's failure, a fixer's work falling back to the writer, still happens on this path.
A non-fixer entry ahead of the fixer entry that fails over the record ends it the same way.
The test row named for a restarted driver calls `entry.Gate()` directly, so it does not exercise this path.

### [MEDIUM] A judge whose start errored still spends bounce budget

**Location:** `internal/shedadapters/bouncer.go:2226`

**Issue:** `RunGated` converts only an `ErrNotStarted` start into a result.
Every other start error, such as a spawn the host refused for EAGAIN, comes back as `runErr`.
`judgeCall` sends `runErr` through `degrade`, which returns a plain `Stuck`, and the engine counts that as a bounce.
Such a judge never came up and gave no verdict, so this is the board entry's defect on a path the diff leaves alone.

### [MEDIUM] The merge-danger blast radius names only shed

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** `boxCapture` decides every send-readiness, idle and interrupt reading in shuttle, through five call sites, and the Claude engine's box reader decides tender revive.
The diff also changes reed's OOM notices and the gate fixer await that webster's fixers share.
A blast radius of `shed` names less than the diff can break.

### [LOW] The interrupted note says no replacement ran after replacements ran

**Location:** `internal/gatefixer/rowgate.go:49`

**Issue:** Past the replacement bound, `waitReplacingInterrupted` returns the interrupted outcome after one or more replacements ran.
The note still says no replacement ran, and the cap test asserts two spawns beside it.

### [LOW] A failed replacement spawn is dropped silently

**Location:** `internal/gatefixer/rowgate.go:287`

**Issue:** The error is neither logged nor put into the findings, and its transient mark is lost.
A first spawn that fails does put its error into the findings.
No row covers this branch.

### [LOW] The orphan sweep can delete the record goneStrandReason reads

**Location:** `internal/gatefixer/await.go:118`

**Issue:** After the reconcile stop removes the strand from reed state, a later shuttle Start's orphan sweep deletes the interrupted run's directory once it is past twice the startup timeout.
A fixer waited on after that reads `died`, spends budget and gets no replacement.
The timing turns on when run.json was last written, and in the incident the record survived, so this is a risk rather than the observed failure.

### [LOW] The replacement's markSpawned is unchecked

**Location:** `internal/gatefixer/rowgate_test.go:556`

**Issue:** Mutation survivors at `rowgate.go:291-292` show that no test fails when the replacement is left out of the wrap's spawned set.
Leaving it out makes `Expired()` true and finalizes the writer's gate early.

### [LOW] The box-disagreement warning fires on every poll

**Location:** `internal/shuttleengine/readiness.go:94`

**Issue:** While the disagreement lasts, every `boxCapture` call logs a Warn with a twelve-line tail, each poll tick of a send-ready wait included.

### [LOW] FixVerifyAwait's doc states the old order

**Location:** `internal/websterengine/verifyfixer.go:246`

**Issue:** The doc says the timeout is checked before a gone strand and names only `DeadReasonDied` for that case.
`gatefixer.Await` now reads the strand first and can return `DeadReasonInterrupted`.

### [LOW] Two new branches have no row

**Location:** `internal/gatefixer/await_test.go:115`

**Issue:** No row covers `goneStrandReason` with a missing run record, and none covers `boxCapture` with a failing plain capture.

### [NIT] Wrapped API-error prompt and empty StartError are untested

**Location:** `internal/shuttleengine/claudeengine/session_test.go:968`

**Issue:** `holdsOnlyAPIErrorPrompt`'s doc claims a wrapped prompt matches, but no row wraps one.
`judgeDiedError`'s not-started branch with an empty `StartError` has no row either.

## Rejected

- code-scope F1 (OOM notice journal lag and wrong process left unbuilt): `paneOOMNoticeLine` already handles a journal line that lands after the kill is counted, naming only the entries that follow those already told and counting the kills when the journal holds fewer. The diff fixes the shared-cgroup double notice. A descendant cgroup and a scope name reused after pid wrap are the writer's own additions beyond the board entry's text, so the entry's ask is met.
- `Await` with `StrandLive` erroring past the timeout: before the timeout it already errored, and `rowWrap.wait` turns the error into a stop and `died`. The only change is `died` in place of `timeout` past the timeout, which is no defect.
- The new `interrupted` dead reason against the `two-way` door: an older binary reads the record through `internal/state`, sees a terminal status and counts the record as spent, so its behaviour degrades but nothing breaks. The value is new content in an existing field, not a format change.
- The readiness commit's Root-Cause trailer: the seat's error quoted a plain pane tail showing the box, and the fallback reads that plain capture. The trailer names the defect the diff removes.
- `planPaneOOMNotices` skipping a cgroup whose first strand's read failed: a failed read that still returns a cgroup fails for every strand in it, and one that returns no cgroup marks nothing, so no notice is lost.
