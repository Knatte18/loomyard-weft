---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: D12 makes lyxcwd read the .git entry at cwd and refuse a cwd without one, which breaks every subpath-anchored hub, including D11's own backend shape
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:52
    summary: D2's premise that every git process hubforge spawns carries the fixture marker is false; hubforge builds hubs through production fabriccli.CloneAndWire, so hub-building git is counted as code under test
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D7 has every Build* return the prebuilt binary, so BuildWithLDFlags silently drops its linker flags; reedcli's switch-seed smoke test builds with Channel=dev and would get the default channel
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: D11's CopyHub and SharedHub build hubs with real git spawns but are not banned from untagged files; the tier-purity scan bans only hubforge.NewHub, and no Decision bounds the new route
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:124
    summary: A misclassified writer on SharedHub corrupts the template for the rest of the binary, and CopyHub copies the corrupted template unchecked; the record bounds only who gets blamed
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:126
    summary: D11's closed definition of a write misses in-process CLI verbs that reconcile and commit, and git reads such as status that rewrite the index, so the plan's reads-only classification rests on an incomplete list
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:15
    summary: Scope omits packages that Decisions change, namely shuttleengine/claudeengine for D7's deny, loomcli, reedcli, orchcli and cmd/lyx runners for D10, and reedcli's watchdog recognizer for D8
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:149
    summary: D13's allowlist does not match PATTERN-shuttle-stop's exemption list as it claims, and a scan for the literal RemoveStrand( misses method-value references, which loomcli already uses, an escape the bound does not name
  - id: F9
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:93
    summary: D7's bound says the agent deny closes a forged prebuilt-binary variable, but the deny it joins is documented in code as a guardrail that a script, bash -c or a shell variable passes
  - id: F10
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:84
    summary: D6's bound omits packages that never call tmuxkit.Main; a tier-1 test that reaches tmux through production code passes the token scan and starts a server in the operator's default socket directory with no check
  - id: F11
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:143
    summary: D12 says GIT_DIR and GIT_WORK_TREE are not honoured but not what that lets through; with either set, in-process geometry and the surviving gitexec spawns, which inherit them, answer for different repositories
  - id: F12
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:144
    summary: D12 calls gitrepo's dotgit read the one local read beside go-git, yet its first bullet adds a second .git parser in lyxcwd
  - id: F13
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: The CopyHub verification scan for the template root falsely matches when the copy's own path has the template root as a prefix, such as hub and hub-copy under one directory
  - id: F14
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:104
    summary: D8's rationale quotes a goal, every test waits for the git and lyx children it starts, that the Goal section does not contain
---

### [BLOCKING] D12's lyxcwd rule breaks subpath-anchored hubs

**Location:** `_lyx/discussion/decision-record.md:136`, and the criterion at `:207`.

**Issue:** D12 has `lyxcwd` find the worktree root "by reading the `.git` entry at cwd", and refuses a cwd with no usable `.git` entry "as today, with the same message".
Today `resolveCore` (`internal/lyxcwd/lyxcwd.go:109`) runs `rev-parse --show-toplevel`, which walks up from cwd, and then reads the recorded anchor (`readRecordedAnchor`, `internal/lyxcwd/anchor.go`).
In a hub with a recorded subpath anchor, the acting cwd is `<worktree>/<anchor>`.
`checkCwdAnchorGate` requires exactly that directory, and it has no `.git` entry.
Under D12 every `lyx` command in such a hub is refused.
The Auto-mode assumptions start D11's shape set with anchor `backend`, so the fixtures this task builds include the broken case.
The "same message" claim is also wrong.
Today a subdirectory resolves and then fails the anchor gate with `ErrCwdOutsideAnchor`, not `ErrNotAGitRepo`.
The acceptance criterion at `:207` tests a worktree root, a linked worktree and a relative `gitdir:` file, but never an anchored subpath, so the plan would ship the regression green.

**Fix:** State in D12 that `lyxcwd` walks up from cwd to the nearest `.git` entry, as `--show-toplevel` does, and that the anchor gate is unchanged.
Add an acceptance criterion: `lyxcwd.Resolve` at a subpath anchor's directory spawns no git and returns the same `Location` as today.

### [BLOCKING] D2's fixture marker misses hub-building git

**Location:** `_lyx/discussion/decision-record.md:52`

**Issue:** D2 says every git process `gitkit` or `hubforge` spawns carries the marker, "set by both packages' spawn helpers", and that git spawned through `gitexec` counts as code under test.
`hubforge.NewHub` builds its hubs through `fabriccli.CloneAndWire` (`internal/hubforge/hub.go:236`), which spawns its clone, worktree, commit and config git through production `gitexec`.
Only `SeedConfig` (`internal/hubforge/seed.go:50`) goes through a `gitkit` helper.
So the bulk of hub-fixture git is unmarked and lands in the code-under-test column.
The board names this split as the measurement's first job, and D3's before-state is captured while every hub test still clones and wires its own hub.
That before-state would overstate code-under-test git by exactly the fixture cost D11 removes, so the before and after comparison misattributes its own headline saving.
D2's rationale, labelling git "without touching production code", is also the reason the marker cannot reach `CloneAndWire`'s spawns.

**Fix:** Decide how hub-construction git is attributed.
One option is to set the marker in the process environment for the duration of a hub build, which is safe only if `NewHub` is serialized or run before parallel tests start.
Another is to classify trace2 events by the fixture's directory: `NewHub` builds under a known root.
State the chosen mechanism and its bound in D2, and add a criterion that a git process spawned inside `NewHub` counts as fixture git.

### [MEDIUM] D7 makes `BuildWithLDFlags` ignore its flags

**Location:** `_lyx/discussion/decision-record.md:89`

**Issue:** D7 says that when the variable is set, "every `Build*` returns it".
`lyxbin.BuildWithLDFlags` exists so that a caller can stamp a different binary.
`internal/reedcli/smoke_switchseed_test.go:38` builds with `-X .../buildinfo.Channel=dev`.
Handed the prebuilt binary, that test runs against the default channel and either fails or stops testing what it names.

**Fix:** State in D7 that only the default-flag builds (`Build`, and `BuildInto` with empty ldflags) return the prebuilt binary, and that a build with non-empty ldflags always builds, once per test binary and per ldflags value.
Add that case to the D7 criterion.

### [MEDIUM] D11's new hub constructors bypass the tier-purity ban

**Location:** `_lyx/discussion/decision-record.md:123`

**Issue:** `cmd/lyx/tierpurity_test.go` bans `hubforge.NewHub` in untagged files because it drives a real clone.
D11 adds `CopyHub` and `SharedHub`, which build the template with `NewHub` plus `AddPair` on first use and then copy or share it.
Neither name matches the banned token, so an untagged test can call `SharedHub` and spawn the full hub build.
D11 states no bound on this new route, and Scope's `cmd/lyx` scans list only the tmux and `RemoveStrand` changes.

**Fix:** Have D11 extend the banned-token set to the new hubforge constructors, or ban `hubforge.` in untagged files outright, as `lyxbin.` already is.
Name the change under Scope's `cmd/lyx` scans.

### [MEDIUM] A write to the shared template contaminates later copies

**Location:** `_lyx/discussion/decision-record.md:124`, and Open risks at `:184`.

**Issue:** `SharedHub` returns the template itself, and its fingerprint check runs only in the sharing test's cleanup.
When a misclassified test writes into it, every later `SharedHub` caller in that binary fails.
Worse, every later `CopyHub` relocates the corrupted template and passes its own checks, because the verification scan looks only for the template root.
Tests on copies then run against a silently wrong fixture.
The open risk names only blame landing on the wrong sharer.

**Fix:** Have `CopyHub` compare the template's fingerprint against the build-time one before copying, and fail naming the fixture.
Alternatively, state in D11 that a fingerprint mismatch marks the template poisoned for the rest of the binary, so every later call fails loudly.
Record the chosen bound in D11 and adjust the open risk.

### [MEDIUM] D11's definition of a write is incomplete

**Location:** `_lyx/discussion/decision-record.md:126`

**Issue:** The plan classifies every hub-taking test against D11's closed list: fabric mutating verbs, git writes, `SeedConfig`, `AddPair` and runs of the `lyx` binary.
Two classes of write fall outside it.
First, in-process module verbs that call `hubreconcile.Ensure` write and commit without being fabric verbs or binary runs: `loomcli` arm, `orchcli` and `battencli` (`internal/loomcli/arm.go:134`, `internal/orchcli/cli.go:118`, `internal/battencli/arm.go:276`).
Second, git reads such as `status` rewrite `.git/index` when the stat data needs a refresh, and the fingerprint's sizes and mtimes see that write.
A test the list classifies as reads-only then fails the fingerprint check, so the plan learns of each case only one red run at a time.

**Fix:** Define a write by its effect: anything that may change a file under the hub, including index refreshes.
Classify any in-process CLI verb and any git subcommand other than in-process go-git reads as a write unless the plan proves otherwise.
Alternatively, exclude `.git/index` stat-only refreshes from the fingerprint and say so.

### [MEDIUM] Scope omits packages that Decisions change

**Location:** `_lyx/discussion/decision-record.md:15`

**Issue:** Scope's In list does not name the following:

- `internal/shuttleengine/claudeengine`, where D7's deny is added. `rawGoCommandPattern` and `steerRawGoDeny` in `settings.go` name the slot and strand variables, and the steer text must change too.
- `loomcli`, `reedcli`, `orchcli` and `cmd/lyx`, whose per-package `lyx` runners move onto `lyxbin.Run` under D10.
- `reedcli`, whose `linuxIsWatchdogDaemon` (`internal/reedcli/smoke_proctree_test.go:84`) D8 reuses, per the plan-writer note.
  That helper is in a `_test.go` file, so `tmuxkit` cannot import it.
  Reuse means moving it into the kit and deleting the copy, or `PATTERN-testkit`'s no-duplication rule is breached.

D12's `gitoracle` rows sit under `internal/gitrepo` and are covered.

**Fix:** Add these packages to Scope with the change each one takes.
State in D8 that the watchdog recognizer moves into `tmuxkit` and that `reedcli` uses it from there.

### [MEDIUM] D13's allowlist and match shape

**Location:** `_lyx/discussion/decision-record.md:149`, and the bound at `:150`.

**Issue:** D13 says its allowlist "matches the entry's exemption list".
`PATTERN-shuttle-stop` exempts only `loomcli` and `orchcli` beside its owner, `shuttleengine`.
D13 also allows `reedengine`, `reedcli` and `shuttlefake`, and it does not say that the entry's text changes to name them.
The scan also matches the literal `RemoveStrand(`.
`loomcli` already takes the method as a value with no call parenthesis (`internal/loomcli/driverlaunch.go:94`, `internal/loomcli/start.go:525`).
Any package can stop a shuttle strand through `f := reed.RemoveStrand; f(...)`, and the bound names only "whose strand", not this escape.

**Fix:** Either amend the entry to list every allowed package, or drop the "matches" claim.
Match the identifier `RemoveStrand` as a selector rather than as a call, or name method-value references in the bound as what the scan lets through.

### [LOW] D7's bound overstates the agent deny

**Location:** `_lyx/discussion/decision-record.md:93`

**Issue:** The bound says the deny "closes" a forged value for spawned sessions.
The deny D7 joins is documented at `internal/shuttleengine/claudeengine/settings.go:71` as matching static text only: "a guardrail and not a barrier".
A script, `bash -c` or a shell variable passes it.

**Fix:** Restate the bound as what passes: a variable set indirectly by a spawned session, plus any operator shell.
State why a forged binary is acceptable under that bound, or add the cheap stronger check: the prebuilt binary's build stamp matches the module's HEAD.

### [LOW] D6's bound omits packages that never call `Main`

**Location:** `_lyx/discussion/decision-record.md:84`, and D8's bound at `:103`.

**Issue:** D6's runtime check runs only inside `tmuxkit.Main`, which only tagged packages call.
A tier-1-only package has neither the check nor `TMUX_TMPDIR` isolation.
The static scan matches tokens, so a tier-1 test that reaches tmux through a production call, such as `reedengine` starting a server, passes it.
The tier-purity file admits this kind of transitive escape for `DeltaGit`.
Such a server lands in the operator's default socket directory.
D8's bound covers its own scan with "tier 1 spawns nothing", but D6's bound names only `TMUX_TMPDIR` and non-tmux binaries.
Tier-2 packages are covered: `PATTERN-test-isolation` makes every package with an `integration` file run through `Main`.

**Fix:** Add the tier-1 transitive case to D6's bound, as the escape the token scan accepts.

### [LOW] D12's `GIT_DIR` bound does not say what it lets through

**Location:** `_lyx/discussion/decision-record.md:143`

**Issue:** With `GIT_DIR` or `GIT_WORK_TREE` in the environment, the in-process geometry and ref reads follow cwd, while every surviving `gitexec` spawn inherits the variables and follows them.
One command then reads one repository and writes another.
The record says only "not honoured".

**Fix:** State the outcome and the bound, for example: `lyx` refuses to run when either variable is set.
Alternatively, record why mixed answers cannot arise, such as `lyx` never running under a git hook environment.

### [LOW] D12 adds a second local `.git` reader while calling gitrepo's the only one

**Location:** `_lyx/discussion/decision-record.md:144`, against `:136`.

**Issue:** The `PATTERN-gitrepo-client-boundary` refinement names `gitrepo`'s dotgit read as "the one local read beside" go-git.
D12's first bullet gives `lyxcwd` its own `.git` and `gitdir:` parsing.

**Fix:** Have `lyxcwd` and `gitrepo` share one reader, in whichever package the import rules allow.
Otherwise, name both readers in the refinement.

### [LOW] CopyHub's verification scan can false-match on a path prefix

**Location:** `_lyx/discussion/decision-record.md:123`

**Issue:** The copy is "verified to contain the template root nowhere".
When the copy's path has the template root as a string prefix, such as `<tmp>/hub` and `<tmp>/hub-copy`, every rewritten path matches and the copy is refused.
The text rewrite has the same prefix hazard in reverse.

**Fix:** State in D11 that the template and its copies live under sibling directories that are not prefixes of one another, or that matching is bounded by a path separator.

### [NIT] D8 quotes a goal the Goal section does not contain

**Location:** `_lyx/discussion/decision-record.md:104`

**Issue:** The rationale cites "the goal's 'every test waits for the git and lyx children it starts'".
The Goal says "a package fails when a test leaves a child running".

**Fix:** Quote the Goal's wording.

## Verdicts on focus items without a finding

- D8's watchdog allowance lets through only the failure, not the process: the watchdog is still killed, so nothing outlives the package.
  A process that merely names `reed watchdog` in its argv is also killed.
  Not a defect.
- D12's 38% figure comes from board entry 246's census (`rev-parse` 18,825, 38%).
  It is motivation, not a premise: D12 rests on the Goal's "fewer git processes in every `lyx` command", and it stands even if the figure is wrong.
  Not a defect.
- Goal against criteria: every Goal bullet has a Decision and a criterion.
  "A package fails when a test leaves a child running" is covered by D8 and the criterion at `:201`.
  No criterion tests something no Decision states.
- Open risks and Auto-mode assumptions do not contradict any Decision's bound.
  Each mitigation a risk names is present in a Decision, apart from F5's contamination gap.
- D11's closed shape set and its write definition are both in `decision-record.md`, so the plan writer needs nothing from `support-log.md`.
  The write definition's content is F6.
