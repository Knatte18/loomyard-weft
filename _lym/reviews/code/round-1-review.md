---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/machinecli/start.go#runBeamStarts
    origin: lens:code-correctness
    summary: Under a prime with no recorded owner, each owned beam starts through a lym beam start <slug> child whose Prepare step claims the prime for this machine, so the verb claims a prime against D10 and its acceptance criterion.
    fix: Stop the by-slug children from claiming the prime. Add a hidden bool flag to the beam start verb in internal/beamcli/cli.go (startCmd), for example --no-prime-claim, that start() carries into orchcli.BeamUpOptions as a new field; in orchcli's prepare path (internal/orchcli/start.go, the claimPrimeNotes call) skip the claim when that field is set. In internal/machinecli/start.go runBeamStarts, append that flag to the args of every by-slug run (not the --all run, which only runs when the prime is owned here). Rename or extend the row PrimeWithNoOwnerIsNeitherClaimedNorStarted so the fake beamStart asserts the flag is passed for each by-slug start, and add a beamcli or orchcli row asserting that a prepared bring-up under the option leaves prime_owner empty.
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/machinecli/start.go#startWorktree
    origin: assist:assist
    summary: On a terminal, the readable summary that cmd/lym wraps every operator verb in hides each worktree's attach command and skip detail and the hold's machine, subjects and re-run, so the desktop launcher's window never prints what D10 says the verb prints.
    fix: "Shape the start envelope so output.Summary shows the required facts. Give startWorktree a json field named line (one of output's lineKeys) that the verb fills per worktree with its outcome plus its detail, and for a started worktree with no window, its attach command, for example started; attach: <cmd>. Give the hub item a label key the summary reads (rename the shortname json key to name, or add one), and render a standing hold as a plain one-line field on the hub, for example hold: <machine> holds <subjects>; re-run: lym adopt <subjects>. Add a row to TestStart_PrintsAttachCommandsWhereNoTerminalLauncherResolves and TestStart_PrintsAStandingHold that runs output.Summary over the verb's envelope line and asserts the attach command, and the hold's machine, subjects and re-run, appear in the summary text."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/machinecli/start.go#beamStartEntry
    origin: assist:assist
    summary: A worktree whose dobby was refused or failed is reported as started, because beamStartEntry decodes only worktree and error while lym beam start reports a dobby failure in outcome, detail and warnings with no entry error.
    fix: Add Outcome, Detail and Warnings (json keys outcome, detail, warnings) to beamStartEntry. In runBeamStarts, mark an entry whose outcome is dobbyengine.OutcomeRefused or dobbyengine.OutcomeFailed as startFailed with its detail, and copy each entry's warnings onto hub.Warnings prefixed with the worktree. Add a row to TestStart_StartsOnlyWhatThisMachineOwns whose fake beamStart returns an entry with outcome failed and no error, and assert the worktree's outcome is failed and the verb exits 1.
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/machinecli/start.go#runBeamStart
    origin: lens:code-tests
    summary: The child boundary is untested; every start test replaces beamStart, so the decode of the lym beam start envelope, lastJSONLine and the fallback reasons can break with no test failing.
    fix: Split the envelope parse out of runBeamStart into a function taking the child's stdout, stderr and run error and returning the entries or the error; runBeamStart keeps the spawn and calls it. Add one tier0 table test of that parse with rows for a success envelope, a partial-failure envelope carrying both error and result, a refusal envelope with no result, output with no JSON line, and an empty output with a run error, asserting the exact entries or error text.
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/launcher_test.go#TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere
    origin: lens:code-tests
    summary: The OwnedElsewhere and NoRecordedOwner rows cannot fail if addLauncher's owner gate breaks, since their boards make the following catch-up remove any launcher the start wrongly wrote; no dobby-level row covers a beam owned here under a prime owned elsewhere.
    fix: Give each row a board whose owners make OwnsAnything true (a prime owned here for the negative rows, so the catch-up keeps whatever the start wrote), keep the Owner seam returning the row's owner, and assert the launcher stands only for OwnedHere. Add a row whose Owner seam returns here for a beam and whose board records the prime under another machine and the beam under here, asserting the launcher stands after the start and its catch-up.
  - id: F6
    severity: LOW
    class: design
    location: internal/machinecli/start_test.go#TestStart_StartsOnlyWhatThisMachineOwns
    origin: lens:code-tests
    summary: The start tests count windows but never assert which workspace and worktree directories open, and no row covers a per-entry child error, a terminal that fails to open, a workspace that fails to open, or a failed hub followed by a good one.
    fix: Record the terminal directory and title and the workspace path in startCalls and assert their exact values in TestStart_StartsOnlyWhatThisMachineOwns, with the prime and an owned beam in one row. Add rows for a fake beamStart entry carrying an error (worktree failed, no window for it), a terminal opener that returns an error (warning plus the attach command on that worktree), a workspace opener that returns an error (warning, terminals still open), and two hubs where the first child fails and the second starts (second hub started, exit 1).
  - id: F7
    severity: LOW
    class: design
    location: internal/battencli/catchup.go#HubCatchUp
    origin: lens:code-tests
    summary: No test asserts that HubCatchUp fills Launcher with the hub's board directory and recorded shortname, so dropping the wiring silently stops every production catch-up from removing launchers.
    fix: "In the existing TestHubCatchUp (internal/battencli/catchup_integration_test.go), assert that the returned CatchUpDeps.Launcher equals dobbyengine.Launcher{BoardDir: fabricengine.BoardDir(hub), Shortname: <the fixture hub's shortname>}."
  - id: F8
    severity: LOW
    class: consistency
    location: internal/dobbyengine/launcher_test.go#TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing
    origin: lens:code-tests
    summary: The redundancy report lists this test as contained by TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere, but only this test plants a launcher first and asserts the catch-up removes it, which the covering test never checks.
    fix: Keep the test and add the line //testtiming:keep pins the catch-up removing a launcher that stands, which the start test never plants directly above its func TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing line.
  - id: F9
    severity: LOW
    class: design
    location: internal/machinecli/start.go#runStart
    origin: lens:code-correctness
    summary: An unknown or ambiguous shortname is refused with hubregistry's creel text, so the verb's refusal reads creel no hub ... way forward run lym creel inbox, which misnames the verb's context.
    fix: "In runStart, when d.lookupHub fails, refuse with the verb's own message, for example lym machine start: no live hub reads the shortname <shortname>; way forward: run lym machine start with no argument to start every live hub, or run lym creel inbox in that hub once to record it, keeping the wrapped cause after it. Update TestStart_NamedHubStartsThatHubAlone's fixture lookupHub and assertion to the verb's message."
  - id: F10
    severity: LOW
    class: design
    location: internal/desktoplauncher/desktoplauncher.go#Desktop.content
    origin: lens:code-correctness
    summary: The WSL .cmd launcher refuses only a double quote in the lym path, but the path sits inside the double quotes of bash -lc, where the WSL default shell expands a dollar sign, backquote or backslash before bash sees the single quotes.
    fix: In the FormWindowsWSL case, refuse a lym path holding any of the characters double quote, dollar sign, backquote or backslash with the same way forward as the double-quote refusal, and add rows for a dollar sign and a backquote to TestAdd_WritesLaunchersRunningMachineStart's refusal case.
  - id: F11
    severity: LOW
    class: design
    location: internal/desktoplauncher/desktoplauncher.go#Desktop.Remove
    origin: assist:assist
    summary: Add and Remove are not serialized across processes, so a Remove that reads no hub launchers left can delete the all-hubs launcher just after another hub's dobby wrote its own, leaving a hub launcher without the all-hubs one until the next add.
    fix: Hold one file lock for the desktop across Add's two writes and across Remove's hub removal, Shortnames read and all-hubs removal. Take the lock path from internal/lock, under the per-user state directory rather than on the desktop so no lock file shows there, and keep a lock failure an error the callers already log as a warning.
  - id: F12
    severity: LOW
    class: design
    location: internal/desktoplauncher/locate.go#freedesktopDesktopDir
    origin: assist:assist
    summary: No test reaches freedesktopDesktopDir's read of user-dirs.dirs, its $HOME/Desktop fallback or its read-error branch, which decide where the Linux launchers land.
    fix: Make the user config directory and home directory parameters of freedesktopDesktopDir (Locate passes os.UserConfigDir and os.UserHomeDir), and add a tier0 table test over a t.TempDir config directory with rows for a user-dirs.dirs naming the desktop, no user-dirs.dirs (the $HOME/Desktop fallback) and a user-dirs.dirs that is a directory (an error).
  - id: F13
    severity: LOW
    class: consistency
    location: pattern/PATTERN-test-isolation.md:67
    origin: lens:code-affected-set
    summary: tmuxkit.Main now points XDG_DESKTOP_DIR at the run directory so no test writes on the operator's desktop, but the background that lists what Main points there does not say so.
    fix: In pattern/PATTERN-test-isolation.md's Private temp directory bullet, after the XDG_STATE_HOME sentence, add a line saying Main also points XDG_DESKTOP_DIR at that directory on every platform, so no dobby a test starts writes a launcher on the operator's desktop.
  - id: F14
    severity: NIT
    class: consistency
    location: cmd/lym/sandbox_coverage_test.go:63
    origin: lens:code-affected-set
    summary: The machine module's exclusion reason names only the name verb, while the module now also holds start, which spawns lym beam start children, opens windows and writes desktop launchers.
    fix: Extend the machine module's exclusion reason in cmd/lym/sandbox_coverage_test.go so it also names start and why it stays outside the sandbox run.
---

### [BLOCKING] The by-slug beam start claims an unowned prime

**Location:** internal/machinecli/start.go#runBeamStarts

**Issue:** With no prime owner recorded and a beam `pay` owned here, `planStart` skips the prime and `runBeamStarts` runs `lym beam start pay` in the prime.
That child prepares its first target (`internal/beamcli/start.go:24`, `internal/beamcli/cli.go:528`).
The prepare path runs `claimPrimeNotes` (`internal/orchcli/start.go:495`), wired for a beam as for the prime (`internal/orchcli/cli.go:281`).
`claimPrimeFor` (`internal/orchcli/ownerpause.go:104`) writes this machine as `prime_owner` when the board records none.
D10's bound says the verb never claims a prime, and its acceptance criterion says it claims no prime under one with no recorded owner.
The commit message of 5bb2fa43 says the verb claims nothing, and `PrimeWithNoOwnerIsNeitherClaimedNorStarted` passes only because the fake `beamStart` never runs the real child.

### [BLOCKING] The terminal summary hides what the verb must print

**Location:** internal/machinecli/start.go#startWorktree

**Issue:** `cmd/lym/main.go:180` turns on the readable summary for every verb whose output is a terminal, which is where a desktop launcher runs the verb.
`output.Summary` renders a list item from its label and line keys and its notes alone (`internal/output/summary.go:161`).
A hub item has no label or line key, so it prints its plain fields; the nested `worktrees` list and the `hold` object are hidden behind the `--envelope` footer.
The attach command, the skip detail with its way forward, and the hold's machine, subjects and re-run therefore never reach the operator's window.
D10 and its acceptance criteria say the verb prints each attach command and the hold's machine, subjects and re-run.

### [MEDIUM] A failed dobby reads as a started worktree

**Location:** internal/machinecli/start.go#beamStartEntry

**Issue:** `orchcli`'s `startDobbyStep` (`internal/orchcli/start.go:586`) records a refused or failed dobby in the entry's outcome, detail and warnings and lets the orch start go on without an error.
`beamStartEntry` keeps only `worktree` and `error`, so such an entry becomes `started`, opens a window and the verb exits 0, while the dobby that takes the runs up is down.

### [MEDIUM] The child boundary has no test

**Location:** internal/machinecli/start.go#runBeamStart

**Issue:** Every start test hands in a fake `beamStart`, so the decode of `{"result":{"beams":[...]}}` (`internal/beamcli/cli.go:437`), `lastJSONLine` and the reason fallbacks run in no test.
A renamed key in `beamcli`'s envelope would turn every start into a failure with no test failing.

### [MEDIUM] The dobby launcher rows cannot fail on the owner gate

**Location:** internal/dobbyengine/launcher_test.go#TestRun_StartAddsTheDesktopLauncherOfAWorktreeOwnedHere

**Issue:** The negative rows' boards record the prime under another machine or none, so the catch-up that `Run` performs after the start removes any launcher the start wrote.
The assertion reads after both, so an `addLauncher` that ignored the owner would still pass.
The acceptance criterion's case, a beam owned here under a prime another machine owns, has no row at the dobby level.

### [LOW] Window routing and the failure fallbacks are unpinned

**Location:** internal/machinecli/start_test.go#TestStart_StartsOnlyWhatThisMachineOwns

**Issue:** The fixture records workspace paths and terminal directories and asserts only their counts, so opening the prime's directory for every beam would pass.
No row reaches a per-entry child error, a terminal opener error, a workspace opener error, or a later hub going on after an earlier hub's child fails.

### [LOW] HubCatchUp's launcher wiring is unasserted

**Location:** internal/battencli/catchup.go#HubCatchUp

**Issue:** The `Launcher` field is what makes every production catch-up remove the launcher, and `TestHubCatchUp` asserts nothing about it.

### [LOW] The contained catch-up test needs its keep marker

**Location:** internal/dobbyengine/launcher_test.go#TestCatchUp_RemovesTheDesktopLauncherOnceThisMachineOwnsNothing

**Issue:** The redundancy report lists it as contained by the start test.
Its assertions differ: it plants a launcher and asserts the catch-up removes it once nothing is owned and keeps it while the prime is owned here; the start test never begins with a launcher standing.
It pins a side effect the covering test cannot see, so it stays and carries `//testtiming:keep`.

### [LOW] The shortname refusal speaks for creel

**Location:** internal/machinecli/start.go#runStart

**Issue:** `hubregistry.Lookup`'s refusal (`internal/hubregistry/hubregistry.go:87`) opens `creel:` and talks of mail across hubs; the verb prefixes it unchanged.

### [LOW] The WSL launcher under-refuses shell specials

**Location:** internal/desktoplauncher/desktoplauncher.go#Desktop.content

**Issue:** The line is `wsl.exe ... -- bash -lc "<posix run>"`.
WSL runs that command line through the distribution's default shell, which expands a dollar sign, a backquote or a backslash inside the double quotes before bash reads the single-quoted path.
Only a double quote is refused.

### [LOW] Launcher upkeep races across hubs

**Location:** internal/desktoplauncher/desktoplauncher.go#Desktop.Remove

**Issue:** Remove reads `Shortnames` and then deletes the all-hubs launcher, while another hub's dobby can run `Add` between the two.
The all-hubs launcher is then gone while a hub launcher stands, until the next add rewrites it.

### [LOW] The Linux desktop directory read is untested

**Location:** internal/desktoplauncher/locate.go#freedesktopDesktopDir

**Issue:** `TestLocate_ChoosesDirectoryAndFormPerPlatform` replaces both resolvers and `TestParseUserDirs_ReadsTheDesktopDirectory` covers only the parse, so the file read, the fallback and the error branch run in no test.

### [LOW] PATTERN-test-isolation misses the new variable

**Location:** pattern/PATTERN-test-isolation.md:67

**Issue:** The background lists the variables `tmuxkit.Main` points at its run directory, and the diff adds one with an isolation guarantee of its own.

### [NIT] The sandbox exclusion reason is stale

**Location:** cmd/lym/sandbox_coverage_test.go:63

**Issue:** The reason excluding the `machine` module names only `name <name>`, and the module now also holds `start`.

## Focus verdicts

- A prime owned by another machine is skipped naming it and `lym adopt root`, and only owned beams start; a prime with no owner is skipped, but its beams' children claim it (F1). No path runs `lym s`.
- Each started worktree gets a terminal through `termwindow`, and the attach command is set where no launcher resolves, though the terminal summary hides it (F2).
- The hold carries its machine, subjects and, for this machine's hold alone, the re-run; the terminal summary hides it (F2).
- `lym beam start` runs as a waited child named through `selfexec.LymBinary`, with its spawn and end logged. It is a synchronous helper and no run process, so `internal/runproc` registration does not apply: not a defect.
- The dobby adds the launcher at every start for a worktree owned here, a beam under another machine's prime included, and every catch-up site, the start's, a sleep's, a hand pass's and adopt's, removes it once nothing is owned; `internal/beamcli` holds none of it.
- The verb removes each launcher whose hub `hubregistry.Live` no longer lists, and the all-hubs launcher goes with the last hub's (race: F11).
- The `.desktop` escaping and the native `.cmd` are correct; the WSL route under-refuses (F10). The assistant built the changed packages for Linux and for `GOOS=windows`.
- Native `wt.exe` resolves and builds its arguments, and the Linux and WSL arguments are unchanged.
- `PATTERN-refusal-way-forward` binds webster, shed and loom alone, so the verb's refusals are outside it; the dobbyengine import allowlist carries the three new imports; `docs/overview.md` and the doc.go files cover the diff.
- Test economy: F8.

## Rejected

- The code-tests lens's NIT that the verb's refusals for an unreadable record and an unresolved binary have no reaching test: `PATTERN-refusal-way-forward` does not bind `machinecli`, and both refusals are one-line pass-throughs of their cause with a way forward, so a test adds nothing a reader cannot see.
- The assistant's BLOCKING severity for the keep marker: the rubric asks for a keep finding, not a block, so it stands as F8 at LOW.
