---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D3 Rationale, D3a Decision, first Acceptance criterion)
    summary: D3 treats a free lock as proof the daemon exited and the first criterion demands no watchdog process once down returns, but D3a makes the daemon release its lock first and then run a list-sessions and maybe a spawn, so the process outlives the free lock
    fix: In D3's Bound, add one sentence that a free lock means the daemon has left its loop and released the lock, and that the exiting process may still be running D3a's one re-check listing and at most one successor spawn when down returns. In D3's Rationale, replace "a free lock is the daemon's own proof of exit" with "a free lock is the daemon's own proof it has left its loop". Reword the first Acceptance criterion to "lyx reed down of a hub's last session returns with the watchdog lock free, and the exiting daemon process ends after its post-release re-check without spawning a successor".
  - id: F2
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (D5)
    summary: D5 adds a PATH prepend to every Bash command of every Claude agent but states no bound, though it puts every executable in LYX_BIN's directory ahead of the rc's PATH, not only lyx
    fix: Add a Bound paragraph to D5 stating that the statement prepends only the parent directory of LYX_BIN, the same directory reed's pane prelude already puts first in the pane shell, so the Bash tool shadows exactly the executables the pane shell already shadows and no others, and that it adds no directory when LYX_BIN is unset or, on Windows, when cygpath is absent.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D8, creel suite bullet)
    summary: D8 points the creel suite's command -v lyx check at the pane shell of a strand whose command starts no fresh shell, but every creel strand is a plain --cmd claude or --cmd bash strand, which the Scope exclusion says start fresh shells, so the creel check has no strand to run in and the rationale that the check now runs where the guarantee holds is false for creel
    fix: Split the creel suite out of D8's first bullet. Give it its own bullet stating that the creel suite's strands all start fresh shells, so its command -v lyx bullet is reworded to say the check does not hold in creel's strands, with a pointer to the Scope exclusion and to SANDBOX-SHUTTLE-SUITE.md for the agent Bash tool check, and that the LYX_BIN-is-set bullet stays as written. Keep the reed suite on the existing pane-shell wording.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (Goal, #680 bullet)
    summary: The Goal promises the fix in the Bash tool of every Claude agent lyx spawns, while Scope Out excludes the Bash tool of a plain --cmd claude strand, which lyx reed also spawns
    fix: In the Goal's #680 bullet, replace "in the Bash tool of every Claude agent lyx spawns" with "in the Bash tool of every Claude agent the Claude shuttle engine spawns".
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: Two stated bounds have no criterion that would catch their failure, D1's stop write when the server does not confirm gone and D3a's no spawn on a cancelled context
    fix: Add two Acceptance criteria. First, a down whose server teardown does not confirm gone still writes the stop marker, returns its error and waits for nothing. Second, a daemon leaving its loop on a cancelled context spawns no successor whatever its post-release listing.
---

### [MEDIUM] A free lock does not mean no watchdog process remains

**Location:** `_lyx/discussion/decision-record.md`, D3 Rationale, D3a Decision and the first Acceptance criterion.

**Issue:** D3 waits for the lock to be free and calls that "the daemon's own proof of exit".
D3a has the daemon release the lock first and then list sessions and possibly call `reedengine.SpawnWatchdog`, so the process is still alive after the lock frees.
The current code already releases the lock in `watchdogCmd`'s RunE only after `runWatchdogLoop` returns (`internal/reedcli/watchdog.go`), and D3a adds a tmux round trip after that release.
The first criterion, "no watchdog process for that hub", can therefore fail when `down` returns, even though every Decision is carried out as written.
The ordering D3a needs (release, then re-check, so a booting session's own spawn can win the lock) is correct.
The problem is that D3's claim and the criterion do not account for it.

### [MEDIUM] D5's PATH prepend has no stated bound

**Location:** `_lyx/discussion/decision-record.md`, D5.

**Issue:** The rubric's attack-surface rule and the discussion spec's bound rule both require a Decision that adds a routing edge to state what the edge lets through and what bounds it.
D5 changes how every bare command an agent's Bash tool runs gets resolved: every executable in `LYX_BIN`'s directory now comes before the operator's rc PATH, not only `lyx`.
In production that directory is the Go bin dir, which holds other tools.
The bound exists: `internal/reedengine/panebin.go` already prepends `filepath.Dir(exe)` to the pane shell's PATH, so D5 only re-asserts in the Bash tool what the pane shell already resolves.
D5 never states this, so a plan writer gets no limit to hold the change to.

### [MEDIUM] D8's creel wording points at a strand class the creel suite never creates

**Location:** `_lyx/discussion/decision-record.md`, D8's first bullet.

**Issue:** `tools/sandbox/SANDBOX-CREEL-SUITE.md` adds strands only with `lyx reed add --cmd claude` and `lyx reed add --cmd bash`.
The record's own Scope Out says both start fresh shells whose bare `lyx` follows the rc's PATH: "The creel suite's plain Claude and bash strands are such shells."
D8 rewords the creel bullet to "the pane shell of a strand whose command does not start a fresh shell".
The creel suite has no such strand, so the rewritten check has nowhere to run.
D8's rationale, "the check now runs where the guarantee holds", holds for the reed and shuttle suites but not for creel.
The check would fail again for the same reason the re-test reported it.

### [LOW] The Goal's #680 wording is broader than the Scope it ships

**Location:** `_lyx/discussion/decision-record.md`, the Goal's #680 bullet.

**Issue:** "every Claude agent lyx spawns" covers a plain `--cmd claude` strand, which `lyx reed add` spawns.
Scope Out excludes that strand's Bash tool, and D5's rationale narrows "lyx agent" to sessions spawned through the Claude engine.
The departure is stated, but the Goal still promises more than D5 delivers.

### [LOW] Two stated bounds lack a criterion

**Location:** `_lyx/discussion/decision-record.md`, Acceptance criteria.

**Issue:** D1 says `down` writes the stop "whether or not the server then confirms gone", and D3 waits only after a successful `Down`.
D3a says a cancelled context spawns nothing.
No criterion covers either, so a regression in either one would pass every listed criterion.
Every other Decision bound maps onto a criterion.

## Focus verdicts

- D5 shadowing: confirmed as F2.
- D1/D2/D3a orderings: stop then boot is safe, because the cold boot touches the signal only after its session exists (`internal/reedengine/lifecycle.go`, `touchDiscoverSignal` after the boot poll), so the stop-woken cycle either sees the session or exits before the boot's own spawn, which then wins the free lock.
  A truncate before consumption also leaves an ordinary discover.
  Exit during boot is closed by D3a.
  The only mismatch found is F1.
- D4 against D3a: consistent. D3a explicitly runs after D4's idle retire, and D4's Bound names D3a as the successor route, so no ordering leaves a live session without a watchdog beyond the lock-probe instant D3's Bound already states.
- Scope Out against D7/D8 and the Goal: F3 and F4. Every Scope In item has a Decision behind it (the Claude engine `doc.go` sentence via the plan-writer note).
- Criteria against Decisions: every criterion traces to a Decision. F5 covers the bounds that lack a criterion.
- Auto-mode premises: the env-file ordering is listed under Open risks, with the shuttle suite's pre-condition as its detector.
  The child-shell premise is backed by `internal/reedengine/doc.go`, which says the typed send-keys line sources the launch script after the pane shell's profile, and the reed suite's pane-shell check stays as its detector. Not a defect.
- No PATTERN constraint violation found. `Engine.Down` writes only the told path, the lock poll lives in `internal/reedcli` beside `watchdogLockFileName`, and the `agentname` constant adds no import.
- No Decision edits this task's own board entry.
