---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/shedengine/run.go:406
    summary: The entry's first goal point is not built. A failure with a transient class is still persisted as state failed; the diff changes only the halt note's anomaly kind.
    fix: "Make the step's failed persist at internal/shedengine/run.go:406 record a failure whose TransientOf class is non-empty as a retryable stop, not state failed, at least until the driver's re-steps are spent. Carry the change through every reader of the state the new stop reaches (internal/sheddriver/driverules.go decideDrive, internal/loomcli/halt.go failedHalt, the run state machine and the board status). If the parent rules the narrower reading in scope (state failed plus the transient class counts as the retryable stop), leave the code as it is; that ruling is the parent's, not the fixer's."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomengine/anomaly.go#AnomalyTransientStop
    summary: The new kind's doc comment, the loomcli package doc and the reflection template all call a transient-stop halt a network or host failure that the driver re-steps. The kind fires for every transient class, and the driver re-steps only git-transport and github-api; agent-start and host-resources get the loop's one re-step and then go to the tackler.
    fix: "Reword the three texts so they hold for every transient class. In internal/loomengine/anomaly.go on AnomalyTransientStop, internal/loomcli/doc.go on the transient-stop paragraph, and contracts/stencils/friction/friction-template-reflection.md on the transient-stop drop rule, describe the halt as an infrastructure failure that lym classed transient and that a re-step can clear. Say that the loop re-steps it once, that the driver also re-steps a git-transport or GitHub API stop with waits, and that any other class goes to the tackler. Drop the claim that every such stop is a network or host failure."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/parentguard.go#ensure
    summary: The ensure doc comment says a gap between the wall-clock and awake times shows the host was suspended. That holds only where Go's monotonic clock pauses during suspend, and a wall-clock step such as an NTP correction also makes a gap. The repository builds for Windows too.
    fix: "In the ensure doc comment in internal/landingshed/parentguard.go, replace \"so a gap between them shows the host was suspended\" with a weaker claim, for example \"so a gap between them points to a host suspend on a platform whose monotonic clock pauses in suspend, or to a wall-clock step\"."
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomengine/anomaly_test.go#TestClassifyHalt
    summary: No TestClassifyHalt row sets both buildSkew and transient. The precedence the ClassifyHalt doc comment states, build skew before transient stop, is unpinned, so swapping the two cases passes every test.
    fix: "Add a TestClassifyHalt row FailedBuildSkewTransient_BuildSkew with state shedengine.StateFailed, buildSkew true, transient true, wantKind AnomalyBuildSkew and wantOK true."
---

### [BLOCKING] A transient failure is still recorded as `failed`

**Location:** `internal/shedengine/run.go:406`

**Issue:** The amended entry's first point under `## What to do` reads: "Record a failure with a non-empty transient class as a retryable stop, not as `failed` / `producer-hard-failure`, at least until the driver's re-steps are spent."
Its `## Why it is abnormal` names both "the persisted state and the halt note" as wrong.
The diff only changes the halt note's anomaly kind.
`run.go:406` still persists `StateFailed` with the class, and no hunk touches the status write.
The change description (`_lym/landing/summary.md`, `## Design decisions`) keeps `failed` on purpose and argues that the transient class is already the retryable marker.
That is the writer's own reading.
The record has no `Added after Discussion` entry that allows it, and the description does not call it a parent correction.
The halt note itself still opens `loom halted failed` with `state: failed`.

### [MEDIUM] `transient-stop` texts misstate what happens to agent-start and host-resources stops

**Location:** `internal/loomengine/anomaly.go#AnomalyTransientStop`

**Issue:** `failedHalt` sets `Transient: st.Transient != ""` (`internal/loomcli/halt.go:127`), so every class in `internal/shedengine/transient.go` gets the kind, `agent-start` and `host-resources` included.
`networkFailure` (`internal/sheddriver/driverules.go:307-316`) is true only for `git-transport` and `github-api`.
Any other class falls to "a transient error that outlived its re-step: the tackler decides", after the loop's one immediate re-step (`internal/shedverbs/loop.go:463-466`).
Three texts state the same false premise:

- the kind's doc comment says "a network or host failure ... The driver re-steps such a stop";
- the `internal/loomcli/doc.go` paragraph says the same;
- the reflection template's drop rule says "a network or host failure that lym classed transient, which the driver re-steps".

An `agent-start` failure, an agent session that never became ready, is neither a network nor a host failure.
The reflection agent drops these halts on that reasoning.
The template's exception, "a failure classed transient that no retry could fix", still lets it file a persistent one, so the effect is a misleading rule, not a lost signal.

### [LOW] The suspend claim on the wait's end line holds only on some platforms

**Location:** `internal/landingshed/parentguard.go#ensure`

**Issue:** `waited` is the wall-clock difference, because `Round(0)` strips the monotonic reading, and `awake` is the monotonic difference.
The comment states as fact that a gap between the two "shows the host was suspended".
That depends on the platform's monotonic clock pausing in suspend.
A wall-clock step, such as an NTP correction, also opens a gap with no suspend at all.
The log line is useful either way; only the comment's claim overreaches.

### [LOW] Precedence of build skew over transient stop is untested

**Location:** `internal/loomengine/anomaly_test.go#TestClassifyHalt`

**Issue:** The doc comment orders the failed cases: build skew first, then transient, then producer failure.
The new rows pin transient alone, transient with interrupted, and transient on a non-failed state.
No row sets `buildSkew` and `transient` together.

## Rejected

- Merge danger `two-way` / blast radius `loom`: the diff adds a new value to the halt note's `anomaly:` line, not a new key. `internal/haltnote` and `internal/reportengine/stops.go` read that value as an opaque string, so an older binary reads a newer note unchanged. The landingshed change only adds log lines, and the reflection stencil runs in a loom run's Friction-Reflect row, so `loom` covers what can break.
- The parent-lock log lines have no test: these are trace lines that only people read, which the testing rules exempt, and the change description says so.
- The holder-change log line and the awake field go past "start, end and holder": they serve the goal point's stated aim of explaining a long wait, so they are not unrequested scope.
- Reflection cannot tell whether re-steps cleared a halt: the template's inputs already include the loom driver's repair and re-step records, and each failed step writes its own suffixed note through `friction.NotePath`, so a halt that the re-steps never cleared shows as repeated notes at one row.
