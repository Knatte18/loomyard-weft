---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-tmux-conf-boot-source-guard.md
    summary: The pre-spawn-seen-server-ended criterion is claimed covered by a differing-identity row that no planned subtest holds, and the candidate decision itself sits where no unit test can reach it
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/04-reed-session-filter.md
    summary: TestListSessions compares the argv to listSessionsArgv itself, so nothing pins the filter-off-on-windows split or the filter's presence on POSIX
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/02-tmux-conf-path-validation.md
    summary: The acceptance criterion that Windows sources nothing for a non-empty tmux_conf has no covering test; the new boot-validation rows skip on Windows instead of pinning the pass-through
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/02-tmux-conf-path-validation.md
    summary: Three new TestUp_BootValidation rows repeat refusal classes TestTmuxConfPath already pins; the wiring at both boot entries needs one row
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/01-tmux-conf-config-key.md
    summary: The TestFill row reed dropped tmux_conf repeats the generic reed dropped top-level key row, whose fill is key-agnostic
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/plan/04-reed-session-filter.md
    summary: ImpactSummary counts eight call sites, but the card reroutes seven list-sessions sites; ListSessions changes only its doc comment
---

### [MEDIUM] Pre-spawn-seen-server-ended criterion has no real covering test

**Location:** `_lyx/plan/03-tmux-conf-boot-source-guard.md`, the paragraph after `TestBoot_TmuxConf`'s subtests ("The pre-spawn-seen-server-ended criterion is the identity subtest's differing-identity row").

**Issue:** The decision record's acceptance criterion "A boot whose pre-spawn read saw a server that has ended by its spawn sources the file into the server its spawn starts" is mapped to "the identity subtest's differing-identity row".
The card's own list of `TestTmuxConfBootGuard` subtests holds no such row.
The identity subtest pins only that `serverIdentityLocked` trims its answer and returns the empty string on an error.
It never compares two identities.
The comparison that decides candidacy is planned inline in `ensureServerAndSessionLocked`: one read before the boot loop and one after the session is up.
No untagged test can drive that function past the spawn, because `spawnSession` is a real `exec.Command` of the configured binary.
`lifecycle_test.go`'s header says so ("Any real-tmux round trip (ensureServerAndSessionLocked, …) is out of hermetic reach"), and `revive_test.go`'s rows stop at "start tmux".
The tmux-tier subtests exercise only the fresh-hub case (empty before, an identity after) and the join case (the same identity on both sides).
No test covers two different non-empty identities, which is the criterion's case and the one D3 step 1 adds over a bare liveness check.

**Fix:** Move the post-spawn read and the comparison into the guard.
`guardBootSessionLocked` takes the pre-spawn identity in place of the `candidate` flag, for example `func (e *Engine) guardBootSessionLocked(identityBefore, confPath string) error`, then reads the identity again and derives candidacy.
`TestTmuxConfBootGuard` then scripts `display-message` through the fake and gets rows for: a seen server replaced by a new one (claim present), the same server on both sides (no claim), and no server before (claim present).
Update the card's sentence to name those rows.
The change touches only a `plan:` handle's declaration head, so no check constrains it.

### [LOW] The session filter's GOOS split is pinned by nothing

**Location:** `_lyx/plan/04-reed-session-filter.md`, the `TestListSessions` sentence, and the overview's Shared Decision `filter-off-on-windows`.

**Issue:** `TestListSessions` will assert that the one call's argv "is `listSessionsArgv` for the host GOOS".
That compares the production helper's output with itself, so it passes whatever `listSessionsArgv` returns.
It pins neither that POSIX carries `-f` with `reedSessionFilter` nor that Windows carries no `-f`.
The ImpactSummary's claim that "the argv on Windows is byte-identical to today's" is the second of those, and no test pins it either.
`TestSwitchClientVia_IssuesOnlyTheThreeCommands`'s rows will be rewritten to the filtered argv, which leaves the same gap if they are also built from `listSessionsArgv`.
The live `TestBoot_TmuxConf` subtest proves that the filter works on POSIX, but not the Windows half of the split.

**Fix:** Give `TestListSessions`, or the switch test, literal argv rows per GOOS in the shape of `TestServerSpawnArgv`'s `goos` rows.
One `linux` row expects `list-sessions -f <reedSessionFilter> -F #{session_name}`, and one `windows` row expects `list-sessions -F #{session_name}`.

### [LOW] Windows sourcing nothing has no covering test

**Location:** `_lyx/plan/02-tmux-conf-path-validation.md`, the `TestUp_BootValidation` rows that "each skip on Windows in `setup`"; `_lyx/plan/03-tmux-conf-boot-source-guard.md`, "on Windows the call is skipped".

**Issue:** The decision record's acceptance criterion "On Windows a non-empty `tmux_conf` sources nothing" has no covering test.
D7 makes it two behaviors: the key is not validated, and the guard does not run.
The new `TestUp_BootValidation` rows skip on Windows, so neither half is pinned anywhere.
`TestBoot_TmuxConf` is `tmux && !windows`.
`TestServerSpawnArgv`'s `WindowsHasNoConfigFlag` row pins only that the spawn carries no chain.

**Fix:** On Windows, have the surviving `tmux_conf` row of `TestUp_BootValidation` (see F4) set an invalid path and assert, through `notErr`, that the boot is not refused for `tmux_conf`, as `ValidWatchdog_On` does.
That pins the "neither validated" half.
State in card 3 that the guard skip is pinned by the validation pass-through together with the unchanged Windows spawn row.

### [LOW] Redundant tmux_conf rows in TestUp_BootValidation

**Location:** `_lyx/plan/02-tmux-conf-path-validation.md`, rows `TmuxConfRelative`, `TmuxConfMissing` and `TmuxConfGlobChar`.

**Issue:** `PATTERN-test-economy`: each refusal class is pinned already by a `TestTmuxConfPath` row (relative path, missing file, name holding `[`), each asserting that the error names the key, the variable and the path.
What `TestUp_BootValidation` adds is that the refusal is wired into both boot entries before any tmux call.
That is one behavior, and one row pins it.
The test's own precedent for the `shell` refusal is a single row, `ShellUnresolvable`.
The two extra rows also force the fixture-sharing variable the card describes for `TmuxConfMissing` and `TmuxConfGlobChar`.

**Fix:** Keep one row, `TmuxConfRelative`, which needs no fixture file, and drop `TmuxConfMissing` and `TmuxConfGlobChar`; `TestTmuxConfPath` stays the covering test for the classes.

### [LOW] Redundant TestFill row for tmux_conf

**Location:** `_lyx/plan/01-tmux-conf-config-key.md`, "`TestFill` gains the row `reed dropped tmux_conf`".

**Issue:** `PATTERN-test-economy`: `TestFill` already builds a `reed dropped top-level key` row from `fillModules`.
That row drops the first top-level key of reed's template, `tmux: ${env:LYX_REED_TMUX:-tmux}`, which is an `${env:...}` key like `tmux_conf`.
It then asserts that the load takes the template default and that the fill writes the key back.
Fill and load are key-agnostic (`assertFilled` and `wantDefault` read whatever the template holds), so a `tmux_conf` row exercises the same path for a different literal: one row per key.
`TestLoadConfig_EnvOverride`'s new `TmuxConf` assertion already pins that the template carries the key under `LYX_REED_TMUX_CONF`, which is the only key-specific fact the acceptance criterion "`lyx config reconcile` adds the key" depends on.

**Fix:** Drop the `TestFill` row and `internal/configreg_test#TestFill` from card 1's `Edit:`.
Name `TestFill`'s `reed dropped top-level key` row plus the `TestLoadConfig_EnvOverride` assertion as the reconcile criterion's coverage.

### [NIT] ImpactSummary site count is off by one

**Location:** `_lyx/plan/04-reed-session-filter.md`, `**ImpactSummary:**`.

**Issue:** "Eight call sites in one package" does not match the card.
It reroutes seven `list-sessions` sites: `listSessionsVia`, `endSessionByNameVia`, `switchClientVia`, `classifyRecordedSessionLocked`, `Down`, `sessionlessSocketHolderPersists` and the zombie check in `ensureServerAndSessionLocked`.
`ListSessions` gains only a doc-comment change.

**Fix:** Say seven call sites, or drop the tally and name the group ("every `list-sessions` call in the package").

## Focus directive verdicts

- **Card 1, `Main` versus `setEnv`:** not a defect.
  `tmuxkit.Main` calls `setEnv(dir)` before `m.Run()` on every non-Windows run (`internal/testkit/tmuxkit/tmuxkit.go`), so the unset covers every tagged package that `TestMain`s through it.
  On Windows `Main` returns `m.Run()` without `setEnv`, but D7 makes the key inert there.
- **Acceptance criteria against card 3's tests:** the empty-key hermetic marker is the refactored `TestBoot_ReadsNoOperatorConfigAndStartsNonLoginPanes`.
  The no-re-source case is the append subtest after a second worktree.
  `-f /dev/null` on the spawn is `TestServerSpawnArgv`'s POSIX rows.
  On the probe it is untouched code, still guarded behaviorally by the operator-config marker test.
  The gaps are F1 (seen-server-ended) and F3 (Windows).
- **Card 4, session reads:** every production `list-sessions` in the repo is one of the seven sites the card routes, plus `ListSessions` reached by `internal/reedcli` and `internal/pairteardown`; the only session loop in a format is `statusFormatSessions`.
  The tests the card leaves out are unaffected:
  - `endsession_test.go`, `generation_test.go`, `lifecycle_test.go`, `launchscript_cleanup_test.go`, `retiring_test.go` and `spawn_test.go` script `list-sessions` by verb, which the fake keys on `args[0]`.
  - `windowsize_test.go` names `statusFormatSessions` by symbol.
  - `bar_integration_test.go` asserts substrings on a kit server that never carries a `sourced` claim.
- **Card order and shared symbols:**
  - Card 2 creates `tmuxConfEnv` before card 3 uses it.
  - Card 3 creates `tmuxConfClaimedOption`, `reedSessionOption` and `TestBoot_TmuxConf` before card 4 uses or edits them.
  - Card 4 reads the two constants exactly as card 3 declares them: server-scoped claim, session-scoped tag.
  - Card 2's discarded second result of `preflightBootHost` is the input card 3 consumes.
- **Granularity of card 3:** one reviewable unit.
  The spawn chain, the identity reads, the guard and the read-back together implement one boot-time sequence (D3 to D5).
  The test helpers serve only the new tmux-tier test, and the identity read is half of the candidate decision.
  Splitting off either half would leave a card with no testable behavior of its own.
- **Test economy:** F4 and F5.
  `TestTmuxConfBootGuard`'s warn and read-back rows and `TestBoot_TmuxConf`'s parse-error, run-time-error and untargeted-`destroy-unattached` subtests pin different behaviors at different tiers.
  The unit rows pin the warning's content and the read-back's `on` route, which no real file can reach.
  The live subtests pin what tmux applies and the session-gone route.
- **Re-sign arrows:**
  - `preflightBootHost` carries the arrow for its new result.
  - Its callers `ensureServerAndSessionLocked` and `withBootOpLockKeepingZoom`, and card 3's `New`, `serverSpawnArgv` and `ensureServerAndSessionLocked`, keep their signatures and correctly carry none.
  - `Engine` gains a struct field and takes no arrow.
- **Attack-surface statements:** each is present and bounded:
  - card 1's key edge;
  - card 2's tightening refusal;
  - card 3's non-fatal guard status, closed pin set and missing-verb fallback;
  - card 4's hiding of file-created sessions.
- **Verify coverage:** `## verify:` runs `./...` for the build, the three tagged vets, the untagged tests and the integration tests.
  That covers `internal/reedengine`, its external test package, `internal/configreg`, `internal/testkit/tmuxkit` and `cmd/lyx`.
  The `tmux` tier is compiled only, and the order matches the template.
  Cards 3 and 4's `lyx gate test --tags tmux ./internal/reedengine` is the package-scoped route the plan stencil names.
  `verify-module-wide` reads only `go` commands.

Evidence beyond reading:

- `lyx loom validate-plan` exits clean on this plan.
- A probe on tmux 3.6 confirmed three mechanics against a throwaway server whose session carries the spawn chain:
  - Card 3's chained guard (claim, `source-file`, then the two pins) applies the file once.
  - A second guard invocation does not re-source.
  - Both pins read `off` after a file setting them `on`, and `list-sessions -f` with card 4's filter lists only the tagged session.
