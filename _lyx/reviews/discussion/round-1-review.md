---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:39
    summary: D2 claims nothing changes for a spawned daemon, but a detached daemon inherits LYX_LOG_FILE and LYX_LOG_LEVEL, so dropping the discard sends a long-lived daemon's stderr half into an uncapped operator file
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:56
    summary: D4 places the zero-pin rebuild in pinGeometryOptionsLocked, which the attach pre-flight also runs, so a degrading attach with the watchdog on would wipe the session's resize pins
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:128
    summary: D7 states no bound for what -f /dev/null drops, and the Open risks remedy is false since reed.yaml has no key for any operator tmux setting
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:51
    summary: D3's claim that every worktree of a hub already agrees on one tmux binary is unchecked; reed.yaml is per-worktree, and the bound does not say a worktree's own tmux value is silently bypassed
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: D5's bound calls POSIX poll mode transient after D4, but a failed hook install or a failed file watch keeps a watcher polling for its lifetime; the 60-second ceiling is the real bound
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:60
    summary: D4 says Windows keeps its current unset path, but Windows has none; the Acceptance criterion, Windows issues neither, is the accurate statement
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:78
    summary: D6 fixes the limit at 107 bytes for all of POSIX, but macOS, a supported build host, has a 104-byte sun_path and a 103-byte limit
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:88
    summary: D7 resolves the shell in the boot path's pre-tmux block, but leaves open where the attach pre-flight gets the resolved path it pins and what it does when the shell no longer resolves
  - id: F9
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:99
    summary: D8 presents a code-reading attribution of the children as established and decides nothing for a re-measurement that does not confirm it
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:84
    summary: D6 and D7 rationale argue against rejected alternatives that support-log.md already records; D7's never-global constraint must move into the Decision before the narrative leaves
  - id: F11
    severity: NIT
    class: scope
    location: _lyx/discussion/decision-record.md:115
    summary: The Closes #495 constraint and the lint-comments acceptance criterion are not plan-writer requirements; landing writes Closes lines and the lint is a mechanical gate
---

### [BLOCKING] D2's bound for a detached daemon rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:39` (D2).

**Issue:** D2 says a detached daemon's stderr is already the null device, "so nothing changes for a spawned daemon".
That holds for the file descriptor only.
`SpawnWatchdog` (`internal/reedengine/spawnwatchdog.go:55`) builds the command with `exec.Command` and a nil `Env`, so the daemon inherits the spawning shell's environment.
`internal/logger`'s `configureFromEnv` reads `LYX_LOG_FILE` at init and calls `SetOutput` on that file.
Today the verb's explicit `logger.SetOutput(io.Discard)` (`internal/reedcli/watchdog.go:678`) overrides it, since an explicit call after init wins.
With D2 the override is gone, so a daemon spawned by `lyx reed up` from any shell with `LYX_LOG_FILE` set appends its stderr half to that file for the daemon's whole lifetime.
That file is the logger's "operator-chosen, unmanaged, whole-verbosity" artifact with no cap and no retention sweep.
With `LYX_LOG_LEVEL=debug` inherited too, which is the route D2 itself advertises, every tmux spawn of every watcher lands there indefinitely.
The logger doc names live substrate tests and orchestrators as the setters of `LYX_LOG_FILE`, so this is a reachable path, not a hypothetical one.
D2 removes a guard, the discard, and the bound it states does not hold, which the rubric's attack-surface rule makes a finding.

**Fix:** Decide what a detached daemon does with an inherited `LYX_LOG_FILE`, and state it as D2's bound.
For example: keep the discard when stderr is not a terminal or when the process was spawned detached, so only a foreground run gains the stderr half.
Or: state that the inherited file is written and bound it.
Add an acceptance criterion that pins the chosen behaviour for a detached spawn.

### [MEDIUM] D4's call site would also run in the attach pre-flight

**Location:** `_lyx/discussion/decision-record.md:56` (D4).

**Issue:** D4 places the zero-pin rebuild in "the fresh-boot pins (`pinGeometryOptionsLocked`)".
`pinGeometryOptionsLocked` is not boot-only: it is called from the boot path (`internal/reedengine/lifecycle.go:512`) and from `AttachArgv`'s pre-flight (`internal/reedengine/attach.go:128`).
On the attach path, every in-closure degrade returns before `installResizePinsLocked` runs; the package doc records this at `internal/reedengine/doc.go:711-720`.
Today, with the watchdog on, such a degrading attach leaves the existing array, pins included, untouched.
If the zero-pin rebuild goes inside `pinGeometryOptionsLocked`, the same attach clears the array and reinstalls only the signal entry, which drops the session's resize pins until the next focusing apply.
That is a regression D4 does not mention, and the Decision's wording points the plan writer at that function.

**Fix:** Name the call site as the boot path only, after its `pinGeometryOptionsLocked` call, and state that the attach pre-flight keeps its current order.
If the intent is to cover attach too, state the degrading-attach consequence as D4's bound instead.

### [MEDIUM] D7 states no bound for what `-f /dev/null` drops, and its stated remedy does not exist

**Location:** `_lyx/discussion/decision-record.md:128` (Open risks), with D7 at line 86.

**Issue:** `-f /dev/null` drops every setting an operator's `~/.tmux.conf` applies to the shared per-hub server.
Reed pins none of the ones an attached operator depends on, such as the prefix and key bindings, `terminal-features` or `terminal-overrides` (truecolor in agent panes), `escape-time`, `history-limit` and `default-terminal`.
A grep of `internal/reedengine` finds none of them pinned.
D7 itself states nothing about this.
The Open risks line says "`reed.yaml` is the configuration surface", but the `reed.yaml` templates (`internal/reedengine/template_posix.yaml`) carry no key for any of those settings, so the stated way forward gives the operator nothing to set.
D7's rationale, "reed pins every option it depends on itself", covers reed's own needs, not the operator's.
The `default-shell` and `default-command` pins also change how every strand pane starts, and D7 states that change only for the login flag.
The board entry asks for the drop, so the behaviour is intended; the gap is the bound and the way forward.

**Fix:** In D7, state what the server now ignores, meaning the operator's whole `~/.tmux.conf`, and what bounds it.
Then either name a real way forward, such as a `-f` path key in `reed.yaml` or sourcing a file the operator names, or state plainly that no in-reed surface exists and that the operator configures the attaching client only.
Correct the Open risks line to match.

### [LOW] D3's agreement premise is unchecked, and the bound omits the bypassed value

**Location:** `_lyx/discussion/decision-record.md:51` (D3).

**Issue:** D3 bounds the override with "the binaries must agree already".
`reed` is not a hub-wide config module: `configreg` marks only `fabric`, `board`, `gate` and `landing` `HubWide`.
Each worktree's `reed.yaml` takes `tmux` from `LYX_REED_TMUX` at materialization.
Nothing checks that worktrees agree, and two different paths to the same tmux version, such as a wrapper or the benchmark's shim, work today against one server.
So the premise is a stated fact without a check behind it.
The bound also never says what the override lets through: a worktree's own `tmux` value, for example a debugging wrapper, is silently bypassed by the daemon's watchers, and the daemon's `--tmux` is whichever binary the first spawning worktree passed.

**Fix:** Reword the bound as what happens, not what must hold: the daemon's watchers use the first spawner's tmux and ignore each worktree's `tmux` value.
Either accept that silently or log one line when they differ, and state which.

### [LOW] D5's POSIX bound overstates transience

**Location:** `_lyx/discussion/decision-record.md:71` (D5).

**Issue:** The focus question resolves as follows.
With `watchdog: off`, `watchLoop` parks on `ctx.Done()` and never polls (`internal/reedengine/watchloop.go:238-243`), so no poll-mode wakeup exists there, and the premise holds for that case.
"Transient after D4" does not hold in general.
When the file watch fails at promotion, the loop logs once and stays in poll mode for good (`watchloop.go:227`).
The hook install is non-fatal, so a failed `set-hook` also leaves the watcher polling.
A session booted by an older lyx and not yet attached has no signal entry either.
In all three cases the 60-second ceiling is what bounds the cost, and a POSIX resize then heals up to a minute late, as on Windows.

**Fix:** State D5's bound as the ceiling on both platforms, and note that `watchdog: off` parks the loop.

### [LOW] D4's Windows clause disagrees with the code and with its own criterion

**Location:** `_lyx/discussion/decision-record.md:60` (D4).

**Issue:** "Windows and `watchdog: off` keep their current unset path" implies Windows has an unset path.
It has none: `pinGeometryOptionsLocked` returns early on Windows, removing only the signal file when the watchdog is off (`internal/reedengine/windowsize.go:162-169`).
The acceptance criterion "`watchdog: off` issues the unset; Windows issues neither" matches the code, so the criterion is the intended Windows behaviour and the Decision's wording is the one that is off.

**Fix:** Reword D4: POSIX with `watchdog: off` keeps its unset; Windows issues no hook call, as today.

### [LOW] D6's limit is Linux's, not POSIX's

**Location:** `_lyx/discussion/decision-record.md:78` (D6).

**Issue:** D6 applies a 107-byte limit "on POSIX".
That is Linux's 108-byte `sun_path` less its NUL.
macOS, which `docs/overview.md` names as a sandbox build host, has a 104-byte `sun_path`, so a path of 104 to 107 bytes passes the check there and tmux still fails as before.

**Fix:** Decide whether the limit is per-OS or whether macOS is out of scope, and state the choice in D6.

### [LOW] D7 leaves open how the attach pre-flight gets the resolved shell

**Location:** `_lyx/discussion/decision-record.md:88` (D7).

**Issue:** D7 pins `default-shell` and `default-command` "at a fresh boot and in the attach pre-flight", but resolves the shell "with `exec.LookPath` in the pre-tmux block", which is the boot path.
The attach pre-flight does not run that block.
The Decision does not say whether attach resolves again, reads a recorded value, or skips the pin, nor what an attach does when the shell no longer resolves.
The refusal acceptance criterion also omits D7's POSIX-only qualifier.

**Fix:** State where the attach pre-flight resolves the shell and whether an unresolvable shell there refuses, degrades with a warning, or skips the pin.

### [LOW] D8 presents an unmeasured attribution as established

**Location:** `_lyx/discussion/decision-record.md:99` (D8).

**Issue:** D8 says the children "are" the watchers' poll-mode round trips, with the name-repair pass "the rest".
The support log records this as resolved by reading the code (Q10), and the benchmark report says its run did not identify them.
The reading is plausible: the shim wrapped only `--tmux`, the watchers spawned the worktree config's binary, and a poll tick runs several round trips per session.
But no measurement attributes the 85 s per hour yet, and "the rest" has no figure.
D8 also decides nothing for a re-run whose children CPU does not fall.

**Fix:** State the attribution as the working hypothesis that the re-measurement confirms, by the shim's per-subcommand counts and the children CPU.
Decide what the last card records if the figures do not confirm it.

### [LOW] D6 and D7 carry rejected-alternative narrative; lift D7's constraint first

**Location:** `_lyx/discussion/decision-record.md:84` (D6) and line 95 (D7).

**Issue:** "Refusing is the smaller change: a socket directory of reed's own would need `-S`…" and "Per-session pins rather than a reed-written config file…" argue against alternatives that `support-log.md`'s Rejected alternatives already records.
The writer's stencil keeps rejected alternatives out of Decisions.
By the completeness-before-leanness test, D6's sentence carries no requirement that is only there: the constraint that reed keeps resolving the socket as tmux does is already in Scope Out.
D7's case differs.
The requirement that the shell options are session-scoped and never `-g` is stated with its reason only in the support log ("Global `-g` shell options at boot: the first booter's shell would bind every sibling's strand panes"), and `Plan-Write` never reads that file.
D7's "set the session's `default-shell`" implies it, but it does not state it.

**Fix:** Add to D7 that the shell options are set per session, never with `-g`, because one server serves every worktree of the hub.
Then drop the two alternative-arguing sentences from D6 and D7, since the support log already holds them.

### [NIT] Two items are not plan-writer requirements

**Location:** `_lyx/discussion/decision-record.md:115` (Constraints) and line 145 (Acceptance criteria).

**Issue:** "The commit message carries `Closes #495`" is landing's work: `contracts/stencils/landing/landing-template-describe.md` writes one `Closes #N` line per issue the task resolves, and the board entry already lists issue 495.
"`lyx loom lint-comments` passes" is a mechanical gate the verify tiers run, not a behaviour of this task, and its second half repeats the Docs constraint.
The `PATTERN-refusal-way-forward` constraint is not a defect: its second clause, that the new refusals name their way forward and are reached by a test, is a requirement the plan writer needs.

**Fix:** Drop the `Closes #495` constraint and the lint-comments criterion.
Keep the refusal constraint.

## Focus verdicts not raised as findings

- D1, D6, D7 acceptance coverage: each Decision has a criterion that fails without it, including the Debug-absent-at-default half of D1 and the refusals of D6 and D7.
  Not a defect.
- D2's other change, the `Long` help, has no criterion; it is help text, and the Docs constraint covers it.
  Not a defect.
