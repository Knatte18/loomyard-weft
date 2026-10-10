---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: D6 claims to compute the socket path as tmux does, but tmux measures the realpath of its socket directory, so a symlinked TMUX_TMPDIR or macOS /tmp passes the check and still fails
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:91
    summary: D6 rationale says tmux exits 0 past the socket-path limit; tmux 3.6 exits 1 on both new-session and list-sessions
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:73
    summary: D5 backoff rule leaves open how an errored, deferred or degraded-box poll tick moves the cadence
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:78
    summary: D5 bound says a watcher whose hook install fails stays in poll mode for its lifetime, but poll mode keeps probing and promotes once a later attach or apply installs the hook
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:95
    summary: D7's -f /dev/null bound omits that a hub server already running from an older lyx keeps the operator's tmux.conf globals until that server exits
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:46
    summary: D2's second Debug route, start it detached with LYX_LOG_LEVEL=debug, names no real detached-start surface and reaches nothing while a daemon already holds the hub lock
---

### [MEDIUM] D6's socket path is not computed as tmux does

**Location:** `_lyx/discussion/decision-record.md:86`, and the acceptance criterion at line 156.

**Issue:** D6 says the boot path "computes the socket path as tmux does" through `socketDirFromEnv`, which joins `$TMUX_TMPDIR` (else `/tmp`) and `tmux-<uid>` with no symlink resolution (`internal/reedengine/socketfile.go:15`).
tmux resolves the directory with `realpath` before appending the key, and the length that fails is the resolved one.
Verified live on tmux 3.6: with `TMUX_TMPDIR` set to a short symlink pointing at a long directory, tmux reported `error connecting to <resolved long path>/tmux-1000/bp (File name too long)`.
So a symlinked `TMUX_TMPDIR` passes D6's check and still hits tmux's error, which is the failure D6 exists to remove.
On macOS this applies with no `TMUX_TMPDIR` at all: `/tmp` resolves to `/private/tmp`, eight bytes longer, so every key within eight bytes of the 103-byte limit is admitted and then fails.
The acceptance criterion pins the check as "a pure function of directory, key and limit", which fixes the unresolved directory in place.

**Fix:** Decide in D6 that the check measures the directory after symlink resolution (falling back to the unresolved path when resolution fails, as tmux does), and state that the pure function takes the resolved directory as its input.
Otherwise, state the symlink gap as D6's bound.
Add an acceptance row for a symlinked directory.

### [LOW] D6's rationale misstates tmux's exit status

**Location:** `_lyx/discussion/decision-record.md:91`.

**Issue:** The rationale says tmux "exits 0 on every invocation past the limit".
On tmux 3.6, `new-session -d` and `list-sessions` past the limit each print the error and exit 1, verified live.
The conclusion still holds, since the boot path `Start`s the server without waiting and its `has-session` poll reads a connect failure as "not up yet".
The premise as written is false, though, and it is the stated reason no probe can tell the failure from a slow boot.

**Fix:** Reword the rationale to say that the boot loop's probes read the connect failure as a server not yet up, and drop the exit-0 claim.

### [LOW] D5 leaves error, deferral and degraded ticks unclassified

**Location:** `_lyx/discussion/decision-record.md:73`.

**Issue:** The cadence doubles on "ticks whose re-apply issued nothing and observed no change of the live box" and resets on "a tick that applied a layout or saw the box change".
`handleWatchOutcome` (`internal/reedengine/watchloop.go:397`) has three other outcomes:
- an error, which leaves `lastApplied` alone;
- a deferral under a held op lock;
- a degraded query (`BoxIsLive` false), which by design is "not an observation".

D5 doesn't say whether any of these doubles, resets or holds the cadence, so the plan writer has to choose.
A deferral means another op is mutating the session, which argues for a reset.
A repeated error doubling to a minute is a cost bound worth stating.

**Fix:** Name, in D5, the cadence move for each of the error, deferral and degraded-box outcomes, and add those rows to the poll-mode acceptance criterion.

### [LOW] D5's lifetime bound overstates the hook-install case

**Location:** `_lyx/discussion/decision-record.md:78`.

**Issue:** The bound says a POSIX watcher stays in poll mode "for its lifetime when its file watch fails at promotion or its hook install fails".
The file-watch case matches the code: `promotionBlocked` stops the probe for good.
A failed hook install does not: a poll-mode tick keeps passing the probe (`watchloop.go:378`), so the next attach pre-flight or layout apply that installs the array promotes the watcher.
The `watchdog: off` park at `watchloop.go:241` is stated correctly.

**Fix:** Make the hook-install case "until a later attach or apply installs the hook", the same wording the older-lyx case already uses.

### [LOW] D7's bound omits a hub server already booted by an older lyx

**Location:** `_lyx/discussion/decision-record.md:95` and the bound at line 104.

**Issue:** D7 notes that a running server ignores `-f`, but the bound states the outcome only for servers this task's binary starts.
A hub whose per-hub server was started before this task keeps every global option, key binding and hook from the operator's `~/.tmux.conf` until that server exits.
New sessions on that server get the per-session shell pins, but nothing else D7 promises.
The marker-option acceptance criterion boots a fresh server, so it doesn't cover this.

**Fix:** Add to D7's bound that the skip takes effect on the hub server's next start, and that a server already running keeps the operator's config until it exits.

### [LOW] D2's detached Debug route is not a real surface as worded

**Location:** `_lyx/discussion/decision-record.md:46`.

**Issue:** The `Long` help is to name a second route: "start it detached from a shell with `LYX_LOG_LEVEL=debug` set".
The only detached start is `SpawnWatchdog`, reached from `up`, `attach` and `resume`.
A hand-run `lyx reed watchdog &` is not detached in D2's sense, because it inherits the shell's stderr and `LYX_LOG_FILE`.
The detached route also does nothing while a daemon already holds the hub's lock: the new spawn exits, and the running daemon keeps its old level.
The bound needs the help to name the verb that spawns the daemon and the precondition that no daemon is running.

**Fix:** Word the second route as running `lyx reed up` (or `attach`/`resume`) with `LYX_LOG_LEVEL=debug` set while no daemon runs for the hub.
State that an already-running daemon keeps the level it started with.

## Focus departures

None.
Every directive was reached and given a verdict.
D2's handling of an inherited `LYX_LOG_FILE` is stated and bounded, and an acceptance criterion pins the child environment, so it is not a defect; F6 is a separate gap.
D4 names the boot path as the call site and states what a degrading attach does to the resize pins: not a defect.
D7 states the per-session, never-`-g` rule and the attach pre-flight's unresolvable-shell degrade.
D7's Open-risks way forward, hand-sourcing into the running server, is tmux's real `source-file`.
D5's bound diverges in one case (F4), and D8 presents the attribution as a hypothesis and decides the refute case.
