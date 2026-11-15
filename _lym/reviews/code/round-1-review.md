---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/orchcli/cli.go:296
    summary: The orch record under the beam's slug makes pair cancel, pair teardown and lym reed down on that slug end the beam's live orch Claude process on its owning machine, which D9 does not license
    fix: In internal/pairteardown, add runproc.RoleOrch to the roles endOwnedProcesses (teardown.go:697) always spares, beside runproc.RoleTeardown, so cancel.go:166, skip.go:94 and teardown.go:335 leave the orch's provider process alone; the orch's own removal stays with the beam's teardown and with D9's EndSubject. Update the Bound at internal/beamcli/doc.go:423 only if it then needs to name the orch, and correct the summary.md design-decision bullet that says a pair teardown ending the orch matches D9.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/runproc/end.go:101-103
    summary: EndSubject's spare set is too narrow; for the root subject it ends hub-wide HubOwner pushes, the test-map pass and the Slack relay, and for every subject it ends RoleTeardown children that pairteardown always spares
    fix: In runproc.endSubject, spare runproc.RoleTeardown in every pass (the runs' pass and the subject's pass), and in the subject's pass spare every role that serves the whole hub rather than the subject, namely RolePush, RoleTestMap and RoleSlackRelay beside RoleAdopt and RoleSync. Update EndSubject's doc comment and the EndSubject sentence in internal/runproc/doc.go to name the spared roles, and add a HubOwner-subject row with push, test-map, slack-relay and teardown records to TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast asserting none of them is signalled.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/orchengine/owner.go:60-64
    summary: The watcher ends the beam's dobby as soon as it reads a foreign owner, which can kill the dobby inside handOver before ArchiveOwner pushes the local tips origin lacks, and nothing restarts that archive
    fix: Make the watcher's end leave the archive to the dobby. In internal/orchcli/watch.go endSubjectProcesses, end the runs and the subject's other processes but leave RoleDobby running while the dobby is live, since dobbyengine.handOver pauses the runs, archives and exits on its own; only when no live dobby is registered for the subject does the end need no dobby step. Alternatively have EndSubject take a flag that skips the dobby pass and pass it from the watcher. Add a row to the watcher or endSubjectProcesses test pinning that the watcher's end does not signal a live dobby.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/orchcli/watch.go:115-157
    summary: The board entry exports one function that ends every process this machine runs for a subject for the watcher, the dobby's handover and the catch-up to call, but the subject-to-runs selection lives in orchcli's unexported endSubjectProcesses, so runproc.EndSubject alone cannot serve the other callers and the summary's claim that they can call the same function does not hold
    fix: Move the run selection out of orchcli into one exported function in a package the dobby wiring and the catch-up can import without reaching orchcli, for example tendercli beside EndOwnedElsewhere, taking the prime location, the runproc dir and the subject's owner, resolving the runs through boardengine.ReadOwners, runproc.Owners and Owners.AdoptSubject and calling runproc.EndSubject. Have the watcher's c.endSubject in internal/orchcli/cli.go:301 call it, delete endSubjectProcesses, and update the orchcli doc and summary.md to name that function.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/orchcli/watch.go:115-157
    summary: The run selection for a subject owned elsewhere has no test; nothing pins beam versus RootSubject, skipping the subject's own owner, or leaving an owner with an unreadable subject alone
    fix: Add one table-driven test for the subject-level end (endSubjectProcesses, or the exported function F4 moves it to) over a hub fixture with a board holding a beam project, a task under it, a root task and an unreadable parent, and registered owners for each; give the function a seam for the final runproc.EndSubject call and assert the runs handed to it for the prime (root tasks only) and for the beam (its tasks only, never the beam's own owner, never the unreadable one).
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-child-processes.md:58
    summary: The Enforcement bullet still says each end goes through End, EndSparing or EndRole, while the Rule the diff changed now adds EndSubject
    fix: In pattern/PATTERN-child-processes.md line 58, change "each end goes through `End`, `EndSparing` or `EndRole`" to "each end goes through `End`, `EndSparing`, `EndRole` or `EndSubject`".
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/orchengine/owner.go:60-64
    summary: A persistent End failure, such as a SurvivorsError or a failed board or owner read in endSubjectProcesses, now keeps the orch strand up indefinitely on a machine that no longer owns the beam, where the strand stop used to depend on nothing
    fix: In ownerTick, call w.owner.Stop even when w.owner.End fails, then return the end's error joined with any stop error so the next tick retries the end; keep done false while the end failed. Update the FailingEndReturnsTheErrorAndStopsNothing row of TestWatcher_EndsTheProcessesAndStopsTheOrchOfABeamOwnedElsewhere to expect calls end then stop s1, and the ownerTick and orchengine doc lines that say a failed end stops nothing.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/start.go endOwnHolder
    summary: The refusal endOwnHolder returns when the orchRegistered read fails has no test row
    fix: Add a row to TestStart_ResumeOverALiveHolder whose orchRegistered fake returns an error (add a registeredErr field), with resumeErrs {held}, expecting exit 1, an error naming the read's cause and lym beam start, no end and one start.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/orchcli/start.go resumeOrch
    summary: No row makes the resume after a successful end of the own holder fail, so nothing pins that such a failure launches nothing fresh
    fix: Add a row to TestStart_ResumeOverALiveHolder with resumeErrs {held, held} and registered true, expecting exit 1, one end, two starts both resuming the recorded session, and an error naming lym beam start.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/run.go:985-998
    summary: The holder read from Run.SessionHolder through runnerSessionStarter into HolderPID or HolderErr, and the carried count, have no test, and registerOrchProcess's HolderErr branch never reaches its warning under test
    fix: Add a holderErr field to fakeStarter in internal/orchcli/start_test.go and a row to TestStart_RegistersTheLaunchedOrchProcess asserting the warning carries its text; in internal/shuttleengine add a test of Run.SessionHolder over a fake engine that implements SessionHolderFinder and one that does not, plus a run with an empty session id.
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/orchcli/start_test.go:1339
    summary: TestStart_StarterErrorFailsAndSavesNoState is listed as contained but pins what its covering tests never check, that a failed launch spawns no watcher and saves no state
    fix: Add the line //testtiming:keep pins that a failed launch spawns no watcher and leaves the saved state unchanged directly above func TestStart_StarterErrorFailsAndSavesNoState.
  - id: F12
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/runproc/doc.go:8-9
    summary: The list of roles registered under HubOwner leaves out RoleOrch, which the prime's orch now registers there
    fix: In internal/runproc/doc.go line 9, add RoleOrch to the HubOwner role list, naming it as the prime's orch provider process.
  - id: F13
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/orchcli/start_test.go:1351-1352
    summary: The a failed resume row fails with the text no transcript for that session, which reads as the missing-transcript case that now launches fresh
    fix: In TestStart_StarterErrorFailsAndSavesNoState's a failed resume row, change err to a cause other than a missing transcript, such as pane never came up.
---

### [MEDIUM] A cancel or teardown on a beam's slug now ends the beam's orch

**Location:** `internal/orchcli/cli.go:296`, with `internal/pairteardown/cancel.go:166`, `internal/pairteardown/skip.go:94`, `internal/pairteardown/teardown.go:335` and `:697-698`.

**Issue:** `RegisterOrch` writes the orch's record under the beam's slug, the owner `pairteardown.endOwnedProcesses` ends through `runproc.EndSparing`, sparing only `RoleTeardown` and the given roles.
A cancel or owner pause of the beam's landing run, or `lym reed down` on the beam, now ends the operator's live orch Claude process on the machine that still owns the beam.
D9 licenses ending the orch only for a subject another machine owns.
The diff patched the same widening for `lym fabric anchor` but not for these callers, and `summary.md`'s design decision claims the pair-teardown case matches D9.

### [MEDIUM] EndSubject ends hub-wide processes and teardown children

**Location:** `internal/runproc/end.go:101-103`, reached from `internal/orchcli/watch.go:153`.

**Issue:** For the prime, the subject's owner is `HubOwner`, and `endSubject` spares only `RoleDobby`, `RoleAdopt` and `RoleSync`.
Every detached push that is not a teardown's registers under `HubOwner` (`internal/fabricengine/spawn.go` `pushSpawnCommand`), so a root subject owned elsewhere kills pushes serving beams still owned here, plus the test-map pass and the Slack relay.
The doc's own reason for sparing adopt and sync, that they serve the whole hub, applies to these too.
Across every pass, `RoleTeardown` children are ended, which `pairteardown` always spares because only an operator's deliberate end may stop a teardown's remote steps.

### [MEDIUM] The watcher can kill the dobby before its hand-over archive

**Location:** `internal/orchengine/owner.go:60-64` with `internal/orchcli/watch.go:153` and `internal/dobbyengine/handover.go:69-80`.

**Issue:** The watcher's tick and the dobby's probe tick read the owner independently.
When the watcher reads the foreign owner first, `EndSubject` ends the dobby before `handOver` pauses its runs and runs `ArchiveOwner`, which pushes the local tips origin lacks.
No process restarts a dobby for a beam owned elsewhere, so the archive waits on a manual `lym beam start`.
D9's "dobby last" orders the end; it does not require the watcher to cut the dobby's own hand-over short.

### [MEDIUM] The exported subject end is not one function the other callers can use

**Location:** `internal/orchcli/watch.go:115-157`, `internal/runproc/end.go:87`.

**Issue:** The board entry's Exports line names one function that ends every process this machine runs for a subject, called by the orch watcher here and by the dobby's hand-over and the catch-up later.
`runproc.EndSubject` needs its caller to name the subject's runs, and the selection of those runs lives in orchcli's unexported `endSubjectProcesses`, tied to the orch's worktree location.
The dobby's hand-over and the catch-up would each have to rebuild that selection, and `summary.md`'s claim that they "can call the same function" does not hold.

### [MEDIUM] The subject's run selection has no test

**Location:** `internal/orchcli/watch.go:115-157`.

**Issue:** No test reaches `endSubjectProcesses`.
`TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` is handed its run list, so nothing pins which runs a beam or the root selects, that the subject's own owner is skipped, or that an owner with an unreadable subject is left alone.

### [LOW] PATTERN-child-processes Enforcement omits EndSubject

**Location:** `pattern/PATTERN-child-processes.md:58`.

**Issue:** The Rule at lines 34 and 37 now lists `EndSubject`, and the Enforcement bullet still lists only `End`, `EndSparing` and `EndRole`.

### [LOW] A failing end keeps the orch strand up forever

**Location:** `internal/orchengine/owner.go:60-64`.

**Issue:** `ownerTick` returns before `Stop` whenever `End` fails.
A persistent failure, such as an unkillable survivor or an unreadable board in `endSubjectProcesses`, leaves the orch strand up on a machine that no longer owns the beam and reruns the end each tick.
Before the diff, the strand stop depended on no such read.

### [LOW] The orchRegistered read failure has no row

**Location:** `internal/orchcli/start.go`, `endOwnHolder`.

**Issue:** The refusal for a failed read of whether the holder is this worktree's orch is reachable but no test row reaches it; the fake in `TestStart_ResumeOverALiveHolder` always answers a nil error.

### [LOW] A failing second resume has no row

**Location:** `internal/orchcli/start.go`, the second `StartSession` in `resumeOrch`.

**Issue:** Every row either ends after the first `held` or resumes on the second try, so nothing pins that a resume failing after the own holder was ended launches nothing fresh, the focus's "on any path".

### [LOW] The production holder read and carry-over count are untested

**Location:** `internal/shuttleengine/run.go:985-998`, `internal/orchcli/start.go` `runnerSessionStarter.StartSession` and `registerOrchProcess`.

**Issue:** `Run.SessionHolder`'s branches, `Run.CarriedInstructions`, the mapping into `HolderPID`, `HolderErr` and `Carried`, and the `HolderErr` text in the registration warning have no test; every orchcli test replaces the starter with `fakeStarter`, which has no holder error.

### [LOW] A contained test pins what its covers do not

**Location:** `internal/orchcli/start_test.go:1339`.

**Issue:** The redundancy report lists `TestStart_StarterErrorFailsAndSavesNoState` as contained by `TestStart_ResumesTheRecordedSession` and `TestStart_ResumeOverALiveHolder`.
Neither asserts that a failed launch spawns no watcher and leaves the saved state unchanged: the first has no failing rows, and the second returns early on a nonzero exit.
The test is kept and needs its keep marker.

### [NIT] runproc's HubOwner role list omits RoleOrch

**Location:** `internal/runproc/doc.go:8-9`.

**Issue:** The enumeration of the roles registered under `HubOwner` was not amended, though the prime's orch now registers there.

### [NIT] The failed-resume row reads as the no-transcript case

**Location:** `internal/orchcli/start_test.go:1351-1352`.

**Issue:** The row's error text is "no transcript for that session", a plain error that does not wrap `ErrNoTranscript`, so it reads as the case that now launches fresh while asserting a failed start.

## Merged

- F2 merges the code-correctness lens's findings 1 and 3, both gaps in `endSubject`'s spare list.
- F10 merges the code-tests lens's findings 4 and 5, both about the untested production holder read.

## Rejected

- The way forward naming `lym beam start` where D9 names `lym orch start` is no finding: `lym orch` has no `start` subcommand (`internal/orchcli/cli.go:184`), and `lym beam start` is the verb that brings the orch up.
- The `internal/fabriccli/anchor.go` change sparing `RoleOrch` is no scope finding: without it the new orch record under the dobby's owner would end the orch on every anchor move, which the diff's licensed registration would otherwise cause.
