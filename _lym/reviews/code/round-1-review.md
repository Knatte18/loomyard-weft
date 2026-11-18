---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/settings.go:339
    summary: The new python hook commands of both engines run a bare lym from PATH, but PATTERN-child-processes requires a hook naming the lym binary to resolve it through selfexec.LymBinary.
    fix: Resolve the lym binary once per launch in each engine's Prepare through selfexec.LymBinary, or selfexec.ResolveLymBinary over an injectable lookup so tests can pass a stand-in path. Hand the resolved path to the python live-check renderers, claudeengine's pythonAllowedCheck in settings.go:339 and codexengine's pythonAllowedCheck in guards.go:38, turning each constant into a function of that path. Shell-quote the path in the rendered command. Update TestShellGuardDenies_PythonRowAsksTheHubOnAMatch, TestGuardCommand_PythonRowAsksTheHubOnAMatch, the settings golden and any other test that renders the command so they run the stub at the injected path, and keep the no-lym row as a nonexistent injected path. Leave the existing creel, hook-edit and hook-batching commands as they are; they predate this diff.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/hookpython_standalone_test.go:343
    summary: The standalone test's target is not a git repository, so it never reaches the real standalone branch, where ResolveWithin succeeds with an empty Board and the verb must still pick the state directory.
    fix: In TestHookPythonAllowedVerb_FollowsAStandaloneRunsStateDirConfig, make the target look like a repository by creating target/.git as a directory with os.MkdirAll, which spawns nothing and keeps the test tier0. Run the verb from a subdirectory of the target so RepositoryRootOf has to climb. Add one row with no shuttle file in the state directory, removing the state dir for that row or using a second target, that expects exit 1 from the template default, covering the fallback that returns the worktree's own layers at hookpython.go:94.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/hookpython_test.go:451
    summary: No row reaches the LoadConfig failure branch of pythonAllowed, so dropping that error check, which would turn a broken config into allowed Python, passes every test.
    fix: Add a row to TestHookPythonAllowedVerb_FollowsTheHubsDenyPython that writes an unparsable shuttle file, such as one whose deny_python value is an unclosed flow sequence, at the hub's default layer, runs the verb from the prime worktree, and expects exit 1 with assertJSONError on the output. Place it before the rows that write a valid file, or have each row write its own file as the rows already do.
  - id: F4
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/shuttlecli/hookpython.go:84
    summary: shuttleConfigDirs picks the hub layers whenever the Bolt checkout exists, but a standalone-capable launch picks hub mode only when preflight.BoardLymPresent finds the Bolt checkout's _lym, so for a hub damaged at _lym the hook reads other config than the launch did.
    fix: In shuttleConfigDirs, after ResolveWithin succeeds, take the hub layers only when preflight.BoardLymPresent(layout) is true; otherwise fall through to the standalone state-dir lookup. If importing preflight into shuttlecli is not allowed, state the hub test the same way, a stat of the Bolt checkout's _lym directory through fabricengine.BoltDir and lymdirs.LymDirName. Add a row to the standalone test with a sibling _bolt directory and no _bolt/_lym, expecting the state directory's value.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:717
    summary: The Codex paragraph still says the python and publish rows follow their switches, but every Codex launch now installs the python row and only its notice follows deny_python.
    fix: In docs/overview.md line 717, replace "the python and publish rows following their switches" with wording that says the python row is installed in every launch and asks the hidden lym shuttle hook-python-allowed on a match, its notice following deny_python at launch, and that the publish row follows its switch.
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shuttleengine/claudeengine/settings.go:337
    summary: The pythonAllowedCheck doc comment has a broken sentence, a period ending the state-directory line and a lower-case and opening the next one.
    fix: In the doc comment above pythonAllowedCheck in claudeengine/settings.go, change "When no hub holds the worktree, it reads the standalone target's state directory." followed by "and it prints nothing, so the hook's output stays the deny alone." into two whole sentences, e.g. end the first with a period and start the second with "The verb prints nothing, so the hook's output stays the deny alone." Apply the same split to the matching comment in codexengine/guards.go, where the comma joins two unrelated clauses.
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#What changes
    summary: The change description says the verb resolves the worktree and loads the hub layers and that a cwd it cannot resolve always denies, but the diff also falls back to a standalone run's state directory, which it never mentions.
    fix: In _lym/landing/summary.md, add to the hook-python-allowed bullet that a cwd no hub holds reads the standalone state directory derived for its repository root, as a standalone launch does. Reword the fails-closed design decision so it says a cwd with neither a hub nor a standalone state directory denies. Add the orch launch test and the standalone verb test to the Tests section.
  - id: F8
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/hookcmd_integration_test.go:546
    summary: The Claude python hook's documented deny when the project directory cannot be entered has no row.
    fix: Add a row to TestShellGuardDenies_PythonRowAsksTheHubOnAMatch with a stub lym exiting 0 and CLAUDE_PROJECT_DIR pointing at a missing directory, expecting the deny and no lym run. Add a field to the table for the project directory override.
  - id: F9
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/settings_test.go:780
    summary: In TestBuildSettings_DenyToggleMatrix the pythonDeny column no longer drives any assertion, so the python_only rows test nothing the base rows do not.
    fix: Remove the three python_only rows, or keep them only if the test gains an assertion on the deny notice that the pythonDeny column decides; if removed, drop the pythonDeny field when no remaining row sets it.
---

### [BLOCKING] Python hook runs a bare lym from PATH

**Location:** internal/shuttleengine/claudeengine/settings.go:339

**Issue:** `pythonAllowedCheck` in claudeengine (`settings.go:339`) and in codexengine (`guards.go:38`) both run `lym shuttle hook-python-allowed` by name from PATH.
PATTERN-child-processes says a command that names this lym binary for a hook resolves it through `selfexec.LymBinary`, and `internal/selfexec/doc.go` lists hooks among its users.
No commit in the range carries a `Pattern-Departure:` trailer.
A PATH lym older than the launching binary lacks the verb and fails, so the guard denies Python even where the hub allows it.
The lym installed on this machine already behaves that way: the assistant's probe found the verb unknown there.
Earlier hooks also run a bare `lym`, but the constraint makes a new violation in this diff BLOCKING.

### [MEDIUM] Standalone test never reaches the real standalone branch

**Location:** internal/shuttlecli/hookpython_standalone_test.go:343

**Issue:** The test's target is a bare temp directory, so `lymcwd.ResolveWithin` fails and the verb takes the error branch.
A real standalone target is a git repository: `ResolveWithin` succeeds with `Board` empty (`hookpython.go:81-87`), and the cwd may sit below the root.
If `shuttleConfigDirs` returned `hubDirs` whenever `ResolveWithin` succeeded, this test would still pass.
That is the path the parent's accepted F2 fix relies on.
The branch returning the worktree's own layers when no state directory exists (`hookpython.go:94`) has no row either.

### [MEDIUM] Config load failure is untested

**Location:** internal/shuttlecli/hookpython_test.go:451

**Issue:** The hub table covers valid values and a resolution failure, and the standalone table covers valid values only.
Neither reaches the `LoadConfig` error return in `pythonAllowed`.
If that check were dropped, the zero `Config` would have `DenyPython` false and the hook would allow Python on a broken config.

### [LOW] Hub test differs from the launch's mode test

**Location:** internal/shuttlecli/hookpython.go:84

**Issue:** `preflight.ResolveMode`, which standalone-capable launches use, picks hub mode only when `BoardLymPresent` finds the Bolt checkout's `_lym`.
Without `_lym` it degrades to standalone and loads the state directory, as in `burlercli/wiring.go:184` and `webstercli/wiring.go:292`.
`shuttleConfigDirs` takes the hub layers whenever `fabricengine.ConfigDirs` returns a non-empty `Board`, which only needs the Bolt directory to exist.
For a hub damaged at `_lym`, the hook and the launch read different config.
The case is narrow: `ResolveMode` itself calls it a damaged hub.

### [LOW] Codex overview paragraph is stale

**Location:** docs/overview.md:717

**Issue:** The diff updated the Claude paragraph at line 683.
The Codex paragraph still says the python row follows its switch, while `switchInstalled` now installs it in every launch.

### [LOW] Broken sentence in pythonAllowedCheck's doc

**Location:** internal/shuttleengine/claudeengine/settings.go:337

**Issue:** The comment reads "…it reads the standalone target's state directory." followed by "and it prints nothing, …", which breaks one sentence across a period.
The Codex twin joins the two clauses with a comma, but they do not belong to one sentence either.

### [LOW] Change description omits the standalone fallback

**Location:** _lym/landing/summary.md#What changes

**Issue:** The summary describes the verb as resolving the worktree with `ResolveWithin` and loading `fabricengine.ConfigDirs`, and it says a cwd the verb cannot resolve denies.
The diff also reads a standalone run's state directory when no hub holds the cwd (`hookpython.go:79-94`).
So an unresolvable cwd with an existing state directory follows that directory's value instead of denying.

### [NIT] No row for an unenterable project directory

**Location:** internal/shuttleengine/claudeengine/hookcmd_integration_test.go:546

**Issue:** The comment on `pythonAllowedCheck` says a project directory lym cannot enter keeps the deny, and no row of the Claude hook test exercises it.

### [NIT] Dead pythonDeny column in the toggle matrix

**Location:** internal/shuttleengine/claudeengine/settings_test.go:780

**Issue:** Line 789 now asserts the python row unconditionally, so the `pythonDeny` column and the python_only rows decide no assertion.

## Focus directive verdicts

- Live read: the verb loads `shuttleengine.LoadConfig(fabricengine.ConfigDirs(layout), "shuttle")` on every call, the same dirs `lym config --resolved` and the orch wiring use, so a flip takes effect without a restart.
- Failure paths: an unresolved cwd, a missing lym, a failing lym and a config that does not load all deny, as the other guard rows' fail-closed shape does; only the test coverage is short (F2, F3).
- Install and announce: both engines install the python row always and announce it by the launch value; not a defect.
- Orch launch: `wireCLI` already loads the hub's layers, and `TestOrchIntegration_WireCLILaunchFollowsDenyPythonAtTheDefaultLayer` exercises it for both values.
- Parity: both engines render the same check. Claude runs it from `$CLAUDE_PROJECT_DIR`, Codex from the hook's working directory, and the verb resolves both the same way.
- Scope: no unlicensed change; one stale statement remains (F5).
- Test economy: `TestBuildSettings_DenyToggleMatrix` and `TestInstalledGuards_CoverEveryRow` carry `//testtiming:keep` markers whose reasons name what they pin beyond their covers, so no fold finding; F9 is the only residue.

## Rejected

- code-tests T4, an untested stencil-seed skip for this verb: the skip is the shared annotation's behaviour, which `cmd/lym/stencilseedgate_test.go` covers, and a test pinning one command's annotation would be a per-declaration check, not a behaviour test.
- code-tests T2 is merged into F2, whose fix adds the missing-state-dir row.
- The assistant's F3 is merged into F3, and its F1 into F1.
- The assistant's F2 is kept at LOW rather than MEDIUM: it applies only to a hub missing the Bolt checkout's `_lym`, which `preflight.ResolveMode` itself names a damaged hub.
