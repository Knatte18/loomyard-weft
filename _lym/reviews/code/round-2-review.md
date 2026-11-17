---
verdict: BLOCKING
findings:
  - id: F15
    severity: BLOCKING
    class: design
    origin: assist:assist
    location: internal/beamcli/cli.go:503
    summary: The hidden flag name no-prime-claim is a repeated literal, spelled twice in beamcli's startCmd (BoolVar and MarkHidden) and again in machinecli's noPrimeClaimFlag, against PATTERN-code-conventions' one-named-constant rule; a rename in beamcli leaves machinecli's children passing an unknown flag with no test failing.
    fix: In internal/beamcli/cli.go declare an exported constant NoPrimeClaimFlag = "no-prime-claim" with a doc comment saying it is the hidden beam start flag that leaves a prime with no recorded owner unclaimed, and use it in startCmd's cmd.Flags().BoolVar and cmd.Flags().MarkHidden calls. In internal/machinecli/start.go delete the noPrimeClaimFlag constant, import internal/beamcli, and build the by-slug child's argument in runBeamStarts as "--"+beamcli.NoPrimeClaimFlag. In internal/beamcli/start_test.go look the flag up with Lookup(NoPrimeClaimFlag). Leave the literal expected argument strings in internal/machinecli/start_test.go's wantBeamStarts rows as they are, since they pin the child's command line.
prior:
  - id: F1
    status: resolved
    answer: runBeamStarts passes --no-prime-claim on every by-slug child; beamcli carries it through startOptions into BeamUpOptions.NoPrimeClaim, and orchcli's prepare skips claimPrimeNotes under it. The child's ensureDobbies calls EnsureLive with the beam's worktree name alone, so the prime-claim branch is never entered. Pinned by the machinecli row, the beamcli flag test and the orchcli NoPrimeClaim row.
  - id: F2
    status: resolved
    answer: The fix taken differs from the one proposed and holds. Worktrees now sit in a top-level list, each labelled shortname/worktree with a line field carrying skip detail or the attach command. The hub's name label and its line carry the hold's machine, subjects and re-run. Reading output.Summary's item and list rendering confirms each reaches the printed text, and the tests assert it through startSummary.
  - id: F3
    status: resolved
    answer: beamStartEntry decodes outcome, detail and warnings, and runBeamStarts marks a refused or failed dobby as failed with the hub warnings kept; rows for failed and refused dobbies assert exit 1.
  - id: F4
    status: resolved
    answer: parseBeamStartOutput is split out and TestParseBeamStartOutput_ReadsTheChildsEnvelope drives lastJSONLine and every reason fallback with exact entries and error text.
  - id: F5
    status: resolved
    answer: Every row's board has this machine own something, so the negative rows fail if addLauncher ignored the owner. The BeamOwnedHereUnderAPrimeOwnedElsewhere row is present.
  - id: F6
    status: resolved
    answer: Terminal directory and title and workspace path are compared exactly. Rows cover an entry error, a terminal and a workspace that fail to open, and a failed hub followed by a good one.
  - id: F7
    status: resolved
    answer: TestHubCatchUp's new subtest compares HubCatchUp's Launcher to the fixture's board directory and hubforge.TestShortname.
  - id: F8
    status: resolved
    answer: The testtiming:keep line sits directly above the test with the requested reason.
  - id: F9
    status: resolved
    answer: runStart refuses with lym machine start's own message and way forward and keeps the cause; the named-hub test asserts all three.
  - id: F10
    status: resolved
    answer: The WSL form refuses a double quote, dollar sign, backquote or backslash in the lym path or shortname before any write; rows cover each.
  - id: F11
    status: resolved
    answer: The fix taken differs from the one proposed and holds. Add and Remove hold one blocking lock across their critical sections, under the per-user state directory, as hubregistry's hubs.lock already sits. internal/lock has no helper for a lock guarding no tracked file.
  - id: F12
    status: resolved
    answer: freedesktopDesktopDir takes its config and home lookups as parameters, and a table drives the user-dirs read, the fallback and the read-error branch.
  - id: F13
    status: resolved
    answer: PATTERN-test-isolation's bullet now says Main points XDG_DESKTOP_DIR at the run directory on every platform, matching tmuxkit's setTempEnv.
  - id: F14
    status: resolved
    answer: The machine exclusion reason now names start and the children, windows and launchers it touches.
---

### [BLOCKING] The no-prime-claim flag name is a repeated literal

**Location:** internal/beamcli/cli.go:503

**Issue:** `startCmd` spells `"no-prime-claim"` in both `BoolVar` and `MarkHidden`, and `internal/machinecli/start.go:42` declares its own `noPrimeClaimFlag = "--no-prime-claim"` for the child it spawns.
`PATTERN-code-conventions` requires a name that can go stale to be one named constant, declared once and referenced everywhere.
The repository already does this for flags a spawner passes to another module's verb: `fabricengine.PauseSlugFlag` and `boardengine.DeferrableSyncFlag`.
Nothing ties the two packages' spellings together.
machinecli's tests replace `beamStart` and assert their own literal, so a rename in beamcli would make every by-slug child fail on an unknown flag, and no test would fail.
The rule makes every violation of a listed constraint BLOCKING.

## Judgments on the focus directive

- **Prime claim (focus 1).** No path through the verb claims a prime; F1 above gives the trace.
  Later, a beam dobby started under an unowned prime may resume a run whose `battencli` pre-run (`claimPrimeOwner`) records a prime owner.
  That is the existing run machinery the record defers to ("the orchs and dobbies take the runs up"), not the verb, and it is no finding.
- **Readable summary (focus 2).** Settled under F2: attach commands, skip details and the hold's machine, subjects and re-run all reach the printed text.
- **Refused or failed dobby (focus 3).** It is reported as `failed` with the dobby's detail, its warnings go onto the hub, and the verb exits 1.
  `orchcli.beamUp` still starts the orch after a refused dobby, so that worktree's orch is live but gets no terminal and no attach line.
  That is a usability gap below BLOCKING, and an incremental round raises nothing below BLOCKING.
- **Child envelope (focus 4).** The decode, `lastJSONLine` and every reason fallback run in `TestParseBeamStartOutput_ReadsTheChildsEnvelope`; only the `exec` spawn in `runBeamStart` stays untested.
- **Launcher rows (focus 5).** Settled under F5.
- **desktoplauncher (focus 6).** Concurrent Add and Remove from two hubs' dobbies serialize on one blocking `flock` under the per-user state directory.
  The WSL route refuses a double quote, dollar sign, backquote or backslash in the lym path or shortname.
  A single quote passes, which is safe: the posix quoting's backslash is not special to wsl.exe's argument split.
  Every branch of `freedesktopDesktopDir` runs in a test except a home-directory lookup error.
- **Test values (focus 7).** Each new test asserts what it claims to pin. Workspace and terminal values are compared exactly.
  The entry, terminal and workspace failure rows check outcome and warning text, and HubCatchUp's Launcher is compared as a whole value.
- **Pattern text (focus 8).** `PATTERN-test-isolation` and the sandbox exclusion reason match `setTempEnv` and the machine module.

The tier-0 gate passed for machinecli, desktoplauncher, dobbyengine, beamcli and termwindow on this tree (851 passed, 0 failed).

## Rejected

- Assistant F16 (the launcher lock bypasses `lock` and sits outside `.lym`): rejected. `PATTERN-lym-files`' lock rule covers locks of tracked files, under a checkout's `.lym/locks`. This lock guards no tracked file and sits in the per-user state directory beside `hubregistry`'s `hubs.lock`, which is the same precedent. `lock.TrackedLockPath` would invent a checkout lock directory under a directory that is not a checkout.
