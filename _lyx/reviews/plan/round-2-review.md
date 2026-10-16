---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/05-down-waits-for-watchdog.md:5
    summary: "card 5 creates the sub-second constant watchdogDownPoll (100ms) with no placement or annotation, and the daemonwakeups scan fails that literal in internal/reedcli/watchdog.go"
    fix: "In card 5's **Intent:** (the paragraph beginning 'probeWatchdogLock calls lock.TryAcquireWriteLock'), add one sentence: the line declaring `watchdogDownPoll` carries `//lyx:one-shot bounded lock poll of at most watchdogDownWait`, the form `internal/orchengine/watchloop.go` uses for `watchLockRetryInterval`, because `internal/reedcli/watchdog.go`, where the card's watchdog* constants and `watchdogLockFileName` live, is listed in `cmd/lyx/daemonwakeups_test.go` `unboundedWaitFiles` and that scan fails a bare `100 * time.Millisecond` there. Declaring the three new constants in `internal/reedcli/up.go` beside `downCmd`, a file the list does not name, is the admissible alternative; the card names whichever it picks. No target changes, so no overlay run is needed."
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/03-down-writes-discover-stop.md:8
    summary: card 3 makes Engine.Down write into the discover signal file but leaves two doc comments that say only a cold session boot touches it, now incomplete
    fix: "In card 3's **Edit:** list add `internal/reedengine#DiscoverSignalFileName` (doc at internal/reedengine/watchdog.go:30-31, 'A cold session boot touches it') and `internal/reedengine#Geometry` (its DiscoverSignalPath field comment at internal/reedengine/geometry.go:45, 'a cold session boot touches to wake the watchdog daemon'; line 45 lies inside the type's span 8-67 per `lyx quarry resolve`). In card 3's **Intent:** add one sentence that both comments are extended to say `Engine.Down` also writes `DiscoverStopMarker` into the file when it ends the server. Checked with `lyx loom validate-plan --overlay` on card 3 with both glyphs added: no finding beyond the baseline."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/04-watchdog-stop-and-recheck.md:52
    summary: the prescribed watchdogHubIdleCycles comment names a non-last down as a case the idle cycles cover, but a non-last down leaves a live session, so no idle cycle arises
    fix: "In card 4's **Intent:**, replace the clause 'a `down` followed by an `up` while the daemon has not consumed a stop: a non-last `down`, or a daemon whose watch failed to open' with: the three cycles cover a `down` followed by an `up` for a daemon whose watch failed to open and so never sees the stop, and a transient list-sessions error, which `sessionsAreIdle` counts as idle."
---

### [MEDIUM] Card 5's 100ms poll constant meets the daemonwakeups scan unannotated

**Location:** `_lyx/plan/05-down-waits-for-watchdog.md:5`
**Issue:** `PATTERN-daemon-wakeups` puts a bounded wait of under a minute outside the rule, so the 10-second `down` wait and its poll are allowed.
Its guard, `cmd/lyx/daemonwakeups_test.go`, still scans by file: every file in `unboundedWaitFiles`, `internal/reedcli/watchdog.go` among them, fails a sub-second `N * time.Millisecond` literal unless the same line carries `//lyx:one-shot <reason>`.
Card 5 creates `watchdogDownPoll = 100 * time.Millisecond` beside `watchdogDownWait` and `watchdogLockFileName` without naming a file or the annotation.
The natural home is `watchdog.go`, where the guard fails the literal.
Card 5's package gate runs only `internal/reedcli`, so the failure surfaces at the batch or plan-level `cmd/lyx` run, one step later than it should.
The precedent for a bounded retry is `internal/orchengine/watchloop.go`'s `watchLockRetryInterval`.

### [LOW] Card 3 leaves two comments about who writes the discover signal stale

**Location:** `_lyx/plan/03-down-writes-discover-stop.md:8`
**Issue:** After card 3, `Engine.Down` writes `DiscoverStopMarker` into the file `DiscoverSignalFileName` names and `Geometry.DiscoverSignalPath` tells.
The `DiscoverSignalFileName` doc says a cold session boot touches it and the daemon wakes on it; the `Geometry.DiscoverSignalPath` field comment says the same of the boot alone.
Neither comment is in any card's targets, so both go incomplete in the commit that adds the second writer.
Card 3 does edit `internal/reedengine/doc.go`'s discover-signal paragraph, but the two comments sit in `watchdog.go` and `geometry.go`.

### [LOW] Card 4's prescribed idle-cycles comment names a case that cannot occur

**Location:** `_lyx/plan/04-watchdog-stop-and-recheck.md:52`
**Issue:** The prescribed `watchdogHubIdleCycles` comment says the three cycles cover a `down` followed by an `up` while the daemon has not consumed a stop, "a non-last `down`" among the cases.
A non-last `down` leaves a sibling session, so the listing is affirmative and no idle cycle is counted; the tolerance never applies to it.
The cases where the tolerance still matters are a daemon whose watch failed to open, which never sees the stop, and a transient list-sessions error, which `sessionsAreIdle` counts as idle.
A comment written as the card prescribes would give a future reader a false reason for the constant.
