---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/07-hermetic-reed-servers.md
    summary: Card 7's POSIX shell refusal in the pre-tmux block breaks every row of TestEnsureServerAndSession_RevivalTrigger in revive_test.go, which the card neither targets nor mentions
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md
    summary: Shared Decision probe-server-bound rests on a false premise; a probe-started server that reads an operator config which creates a session is never reaped, so reed's new-session joins a server that read ~/.tmux.conf
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/07-hermetic-reed-servers.md
    summary: The new ChainGate row ShellUnresolvable_PinsNothingAndStillChains re-runs the path the existing Status_On_DoesNotSuppress row already runs, since the shared fixture's shell already does not exist
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/07-hermetic-reed-servers.md
    summary: tmuxkit.ConfigText is an export consumed from another package, so the spec's granularity rule gives it its own card ahead of card 7
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/05-poll-mode-backoff.md
    summary: Card 5 removes tickerPeriodFor's last caller and deletes tickerPeriodFor in the same card, where the spec's granularity section asks for two cards
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/05-poll-mode-backoff.md
    summary: Card 5 claims every other loop test passes because a changed or applied tick returns to the base, but TestWatchLoop_OpenerFailureStaysInPollMode runs on unchanged ticks and passes only if the test ceiling stays small
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/plan/02-spawn-watchdog-env.md
    summary: Card 2 says spawnwatchdog.go's file comment states the child's stderr is discarded, but only SpawnWatchdog's doc comment says so
---

### [BLOCKING] Card 7's shell refusal breaks TestEnsureServerAndSession_RevivalTrigger, which no card targets

**Location:** `_lyx/plan/07-hermetic-reed-servers.md`, the boot-path paragraph and the "Tests, untagged" paragraph; `internal/reedengine/revive_test.go:13-86`.

**Issue:** Card 7 makes `ensureServerAndSessionLocked` refuse an unresolvable shell in its pre-tmux block, ahead of `probeCapabilityLocked`, on every GOOS but `windows`.
The card works out the effect of the shared fixture's nonexistent shell (`lock_test.go:58`, `does-not-exist-shell.exe`) on `TestUp_BootValidation` only.
`reviveTestEngine` (`revive_test.go:13-22`) builds on the same `newTestEngine`, keeps that shell, and `TestEnsureServerAndSession_RevivalTrigger` calls `ensureServerAndSessionLocked` directly.
On POSIX, every row then fails on the shell refusal:
`RecordedNotLiveRevivesFirst` wants `errReviveFirst`, the three `...Creates` rows want an error containing "start tmux", and `LiveSessionIsUsedAsFound` wants a nil error.
The test sits in an untagged file, so card 7's own card gate fails, and the fix needs an edit to a file outside the card's targets.

**Fix:** Add `internal/reedengine/revive_test.go` to card 7's `Edit:` group, and state in the card's tests paragraph that `reviveTestEngine` sets `cfg.Shell` to a shell `exec.LookPath` finds (`sh`) on POSIX, as the card already proposes for the `ValidWatchdog_*` rows.
Before fixing, sweep the other untagged callers of `Up`, `Resume` and `ensureServerAndSessionLocked` that run on `newTestEngine` against a fake tmux.

### [MEDIUM] Shared Decision probe-server-bound rests on a false premise

**Location:** `_lyx/plan/00-overview.md`, `### Decision: probe-server-bound`; card 7's "The boot path" paragraph.

**Issue:** The decision keeps the capability probe's `list-commands` without `-f /dev/null`.
Its premise is that the transient server it starts "exits when it has no session", and that a lingering one is the stale-holder case `sessionlessSocketHolderPersists` reaps.
I checked this against tmux 3.6 here: `tmux -L <key> -f <conf> list-commands` starts a server that reads the config, and a `@probe_marker` option the config sets reads back afterwards from `show-options -gqv`.
So the probe's server is a reed-started server that reads the operator's `~/.tmux.conf`.
If that config creates a session, a common `new-session` line, the server holds a session.
`sessionlessSocketHolderPersists` then never reaps it, and the following `new-session -f /dev/null` goes to that running server, which ignores `-f`.
Reed's session then lives on a server carrying the operator's whole config, which is what D7 ("reed's own servers read no `~/.tmux.conf`") and its acceptance criterion rule out.
The decision record has no such bound, so the plan adds a gap to D7's guarantee and records it as a bound on its own authority.

**Fix:** Have card 7 also carry `-f /dev/null` on the probe's server-reaching invocation in `probe.go`, which is harmless against a running server.
The probe is then a server-starting invocation that reads no config, and the Shared Decision either drops or records that the probe carries the flag.
Add `internal/reedengine#Engine.probeCapabilityLocked`, or the probe's argv-building member, to card 7's `Edit:` group, and add a `TestServerSpawnArgv`-style assertion or a row pinning the flag on the probe argv.

### [LOW] ChainGate's new shell row duplicates an existing row's path

**Location:** `_lyx/plan/07-hermetic-reed-servers.md`, the `attach_test.go` sentence; `internal/reedengine/attach_test.go:184-214`.

**Issue:** `newAttachTestEngine` (`attach_test.go:85-87`) builds on `newTestEngine`, whose shell does not exist.
After card 7, every existing `TestAttachArgv_ChainGate` row already runs the unresolvable-shell pre-flight.
`Status_On_DoesNotSuppress` already asserts a chained argv on that path.
The new row `ShellUnresolvable_PinsNothingAndStillChains` re-runs it, and adds only the "no `default-shell` or `default-command` set-option" assertion.

**Fix:** Drop the new row, and add the no-shell-pin assertion to `Status_On_DoesNotSuppress`, or to the `assertStrandOptionsReasserted` check every row already makes.

### [LOW] tmuxkit.ConfigText belongs on its own card

**Location:** `_lyx/plan/07-hermetic-reed-servers.md`, `Create:` entry `plan:internal/testkit/tmuxkit#ConfigText`.

**Issue:** The spec's "Card granularity" section gives an export reused across packages its own card, even when one card is its first consumer.
`ConfigText` is a new `tmuxkit` export, and `internal/reedengine`'s `tmux`-tier test reads it.
Card 7 bundles it, with the `tmuxkit.go` edit, into a card that already carries a large set of targets.

**Fix:** Split a card ahead of card 7 that creates `ConfigText` and edits `internal/testkit/tmuxkit/tmuxkit.go` so `serverConfig` writes it.
Card 7 then uses the export, and its package-doc rewording of `tmuxkit.go` moves with it or stays as an edit on card 7.

### [LOW] Card 5 deletes tickerPeriodFor in the card that removes its last caller

**Location:** `_lyx/plan/05-poll-mode-backoff.md`, the `Delete:` group and the `watchLoop` paragraph.

**Issue:** The spec's "Card granularity" section states that removing a symbol's last caller and deleting the symbol are two cards, not one.
Card 5 replaces `watchLoop`'s ticker, the only production caller (`watchloop.go:286`), and deletes `tickerPeriodFor` in the same card.

**Fix:** Move the `Delete: internal/reedengine#tickerPeriodFor` group, with the `watchloop_test.go` step that tests it, into a card placed after card 5.
Or, if the plan holds that one caller in the same function edit makes the split meaningless, record that in a Shared Decision.

### [LOW] Card 5's reason the other loop tests keep passing is wrong for an unchanged-tick test

**Location:** `_lyx/plan/05-poll-mode-backoff.md`, "Every other loop test keeps passing because a changed or applied tick returns to the base"; `internal/reedengine/watchloop_test.go:932-955`.

**Issue:** `TestWatchLoop_OpenerFailureStaysInPollMode` stays in poll mode with a scripted session that never changes.
After card 5 its ticks double the wait, and the test still wants five `list-panes` calls within 300 ms.
It passes only because `watchdogTestTiming`'s `PollCeiling` stays small, not because a tick returns to the base.
Its doc comment, "keeps polling at the poll cadence", also goes stale, and the card's doc-follow-up covers only `TestWatchLoop_PollModeByDefault`.

**Fix:** Replace the sentence with the real reason: the test ceiling must keep five backed-off ticks inside the 300 ms window.
Name that bound when setting `PollCeiling`, and reword `TestWatchLoop_OpenerFailureStaysInPollMode`'s doc comment to the backed-off poll.
`watchloop_test.go` is already a card 5 target.

### [NIT] Card 2 names a file comment that does not carry the claim

**Location:** `_lyx/plan/02-spawn-watchdog-env.md`, "Its doc comment and the file comment stop saying the child's stderr is discarded"; `internal/reedengine/spawnwatchdog.go:1-9`, `:28-30`.

**Issue:** The file comment at `spawnwatchdog.go:1-9` does not mention stderr.
Only `SpawnWatchdog`'s doc comment, at lines 28-30, says the child's stderr is discarded.

**Fix:** Drop "and the file comment" from the sentence.
