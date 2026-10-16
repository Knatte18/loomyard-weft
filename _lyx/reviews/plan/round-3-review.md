---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/04-shuttle-conditioned-send.md:72
    summary: Card 4's not-landed and lone-stray-character rows run over pickerEngine, which has no InputBoxClearer or StrayInputReader, so no row reaches the keepUnlanded skip in clearUnlandedText or a stray clear, and both rows pass with or without the card's no-clear guarantees
    fix: In card 4's Intent, declare in conditionedsend_test.go a clearingPickerEngine, a pickerEngine that also implements InputBoxClearer (ClearInputSequence returning the single key C-u) and StrayInputReader (StrayInput reporting a box holding exactly one character), and run the lone-stray-character and not-landed rows over it on the typed path. The not-landed row then asserts that tuiReed.clears is zero, that the box still holds the sent text and that the error carries the note the unlanded text was left in the input box. The stray row asserts that tuiReed.clears is zero and the box still holds x. Keep the other rows on pickerEngine and pastePickerEngine. No target changes.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-github-per-request-deadlines.md:31
    summary: Card 3's held-page row leaves page 1 only 30 ms between its 170 ms handler delay and its 200 ms per-request deadline, which token resolution, the pause-record read, loopback HTTP and decoding must fit into under -race on a loaded gate slot, so the passing case can fail intermittently
    fix: In card 3's Intent, change the row's numbers to a per-request budget of 400 ms, a first page answered after 300 ms (100 ms left of a shared deadline when page 2 starts), the gate's Sleep of 150 ms that then lifts the entry, and page 2 answered after 200 ms. With a fresh deadline per page, page 2 then has 200 ms of margin; under a shared deadline, card 2's extension moves page 2's end to 250 ms after it starts, before its answer at about 350 ms, so the row still fails without card 3. Update the margin sentences at lines 31 and 33 to these values. No target changes.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/06-orch-watch-binary-reexec.md:28
    summary: Guard branches that cards 4, 5 and 6 prescribe have no row that reaches them, the round-1 gap whose fixed rows are missing again from the current cards, so a mutant of each branch passes every row
    fix: Add one row per branch to the test each card already plans. In card 4's TestRunner_SendConditioned, a row whose pane shows no input box (no caret in the tuiReed capture) refuses with a SessionBusyError, types nothing and leaves the clock where it was. In card 5's TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads, a row with fakeSession.sessionStateErr set makes the tick return the error, type nothing and keep the notice queued. In card 6's TestRun_ChecksTheBinaryOnlyAtARestartSafeTick, a row in the idle phase tells the watcher, through WatchPause, a pause record whose file holds the text not json; pauseTick reads it as uncovered and logs a Warn, so each tick succeeds, and restartSafe's record read fails, so the check never runs. These are rows in tests the cards already plan, so no target or budget changes.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/06-orch-watch-binary-reexec.md:47
    summary: Card 6's non-idle row saves PhaseHandoffRequested with no phase entry time, so the first tick's handoff timeout (now minus a zero PhaseEnteredAt is past HandoffTimeoutS) returns the watcher to idle and the check then runs, failing the row as written against a correct implementation
    fix: In card 6's Intent, have the non-idle row save PhaseHandoffRequested with PhaseEnteredAt set to the env clock's now and PhaseInjected true, so that no tick times the handoff out or types a handoff request, and the phase stays non-idle for every tick of the row. No target changes.
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-pauserecord-network-held.md:16
    summary: Card 1 again defines held as the time from entry until the wait returned, which is nonzero for every gated call under a real clock, so card 2 would extend every call, and its fake clock advances only in Sleep, so no row catches it; the round-1 fix is missing from the card
    fix: In card 1's Intent, define held as zero unless the wait slept at least once, and say how Network tells, by wrapping the Sleep it hands the waiter in a closure that notes it ran; held is measured from entry to the wait's return only in that case. In TestNetwork, give the uncovered SetGateSlug row's gate a Now that advances on every call, not only in Sleep, and keep its assertion that held is zero, so the row fails an implementation that measures elapsed time unconditionally. No target changes.
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-orch-conditional-notice.md:28
    summary: Card 5 keeps the .notice suffix on purpose, so a watcher built before this change types a conditioned revive notice as a plain one with no state check, breaking D5's at-delivery bound with no stated bound; the round-1 fix is missing from the card
    fix: In card 5, name a conditioned notice <time>-<rand>.when-<state>+<cause>.cnotice, which an old watcher's noticeNames skips, and have noticeNames list both suffixes in arrival order. Add these Edit targets after internal/orchengine#ListNotices - internal/orchengine#noticeName re-signed as func noticeName(now time.Time, when *NoticeCondition) (string, error), internal/orchengine#noticeNames, and internal/orchengine#QueueNotice, which passes nil. Replace the sentence saying an old watcher delivers it as a plain notice with one saying an old watcher leaves it queued until a watcher of this build delivers or drops it, and add a TestQueueConditionalNotice assertion that the conditioned file's name does not end in .notice. The overlay with these three targets passed validate-plan with no new finding.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/12-tender-rename.md:38
    summary: Card 12 says error texts keep a pause prefix, but errEntryGone and errOwnedElsewhere in pause.go start with pauseengine, which its surgical edits leave, so its closing grep claim fails; the round-1 fix is missing from the card
    fix: In card 12's list of surgical edits, replace the sentence on error texts with one saying the errEntryGone and errOwnedElsewhere texts in pause.go, which start with pauseengine, now start with tenderengine, that the cli's error texts keep their pause prefix and the shortcut spellings lyx pause and lyx resume, and that no test asserts on the pauseengine prefix. No target changes.
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/11-reed-attach-exec.md:34
    summary: Card 11 updates reedcli/doc.go but leaves attachCmd's Long, which says attach hands stdio to a tmux attach-session child, false on Unix; the round-1 fix is missing from the card
    fix: In card 11's Intent, beside the reedcli/doc.go sentence, add that attachCmd's Long says attach replaces itself with tmux attach-session on Unix and runs it as a child on Windows, keeping the rest of the Long. No target changes, since internal/reedcli#reedCLI.attachCmd is already on the card.
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/plan/10-tender-revive.md:84
    summary: Card 10 both adds a revive agent-outside-its-own-pair row to refusal-spec.md and makes the existing caller row name revive, so one refuseCaller refusal is documented twice; the round-1 fix is missing from the card
    fix: In card 10's refusal-spec sentence, drop the new row revive agent outside its own pair and keep only the edit that makes the existing caller row (pause or resume, agent outside its own pair) name revive in its title, trigger and way forward; keep the new revive unknown-slug row. No target changes.
---

### [MEDIUM] Card 4's no-clear rows run over an engine that can never clear

**Location:** `_lyx/plan/04-shuttle-conditioned-send.md:72`, with the stray row at line 69 and the engine choice at line 60.

**Issue:** `TestRunner_SendConditioned` runs over `pickerEngine` (`internal/shuttleengine/fakes_test.go:924`), which is `readinessEngine` plus `RewindPickerReader`.
It implements neither `InputBoxClearer` nor `StrayInputReader`.
`clearUnlandedText` (`internal/shuttleengine/send.go:703`) returns before any key when the engine is not an `InputBoxClearer` (`send.go:713-716`), and `clearStrayCharacter` (`send.go:268-272`) declines for the same reason.
So the not-landed row's "no clear key sent" holds whether or not the card adds the `keepUnlanded` skip, and the stray row's "no clear key sent" holds even if `conditionedReading` reuses the idle wait's `busyReading`, which clears a stray character for a capable engine.
The production engine has both capabilities, and the `keepUnlanded` skip is the card's one change to existing send code, so neither guarantee in D5 ("it never clears its own unlanded text", "refuses at once, sending no key, when its ready reading shows … a lone stray character") is pinned by a row that reaches it.
The paste-path row cannot fill this gap: a revive line of 20 or more runes collapses into the paste placeholder in `tuiReed`, which `clearUnlandedText` already declines to clear as not this send's text.

### [MEDIUM] Card 3's held-page row has a 30 ms margin on page 1

**Location:** `_lyx/plan/03-github-per-request-deadlines.md:31`

**Issue:** The row's per-request budget is 200 ms, and the handler answers page 1 after 170 ms.
Page 1's context is derived just before the request, so the 30 ms left must cover `resolveToken`, `pauserecord.Network`'s record read, the go-github request build, the loopback round trip and the JSON decode, all under `-race` in a shared gate slot.
A miss there fails the passing case with a deadline error on page 1, which is a flaky test rather than a caught defect.
The row's ability to tell a fresh deadline from a shared one rests on page 2, where a slower machine only makes the shared-deadline variant fail harder, so the page-1 margin can be widened without weakening the row.
Round 2's F6 set these numbers to widen a 20 ms margin; this finding widens the remaining one on page 1.
The rest of the focus question holds: `callTimeout` and `openPRListTimeout` stay consts, the shortened budget is the `perRequest` parameter of `listOpenIssues`, each page loop derives its own `context.WithTimeout`, and the row's client is `githubclient.NewWithBaseURL` through the `NewGitHubClient` seam, whose `authRT` meets the pause gate, with the gate's `Sleep` asserted to run once.

### [MEDIUM] Guard branches in cards 4, 5 and 6 have no reaching row

**Location:** `_lyx/plan/06-orch-watch-binary-reexec.md:28`; also `_lyx/plan/04-shuttle-conditioned-send.md:39` and `_lyx/plan/05-orch-conditional-notice.md:34`.

**Issue:** Each card prescribes a guard branch that none of its rows reaches:

- Card 4's `conditionedReading` refuses "a pane too short or showing no input box" with a `*SessionBusyError`, and no row shows a pane with no input box.
- Card 5's `deliverNotice` says "a state read error returns the error and keeps the notice", and no row sets `fakeSession.sessionStateErr` (`internal/orchengine/watcher_test.go:52`).
- Card 6 says "A state or record read error is not safe".
  The state half has a row (line 50), but the pause record half has none: the rows cover no record, a live hub entry and a live slug entry only.
  The branch is reachable through `Run`, since `pauseTick` (`internal/orchengine/pause.go:41-55`) logs an unreadable record at Warn and returns not held, so the tick succeeds, and `pauserecord.Read` (`internal/pauserecord/record.go:84-97`) fails on a file that does not decode.
  D8 rests the exec on "no hub pause entry is live at all", and an unreadable record is the one state where a live entry cannot be ruled out.

Round 1's F3 raised these three rows, and its fixer report says each was added, but none is in the current cards, so this is filed once about the method, as the round-1 finding was.

### [LOW] Card 6's non-idle row leaves the phase entry time zero

**Location:** `_lyx/plan/06-orch-watch-binary-reexec.md:47`

**Issue:** `tickHandoff` (`internal/orchengine/watcher.go:826-833`) returns the watcher to idle through `toIdle`, which saves `PhaseIdle`, once `now` minus the phase's start reaches `HandoffTimeout`.
`newWatchEnv` sets `HandoffTimeoutS: 100` with a fake clock in 2026, so a state saved with only `Phase: PhaseHandoffRequested` times out on the first tick.
`restartSafe` then re-reads an idle phase and the check runs on every later tick, so the row "the check never runs" fails against a correct implementation unless the saved state carries a current `PhaseEnteredAt`.
The guard branch itself is reachable through `Run`; only the named input needs the entry time.

### [MEDIUM] Card 1 measures held on every call

**Location:** `_lyx/plan/01-pauserecord-network-held.md:16`

**Issue:** The Intent says `held` is "the time from its entry until the wait returned, measured on the gate's clock".
Under the real clock that is a nonzero duration for every call the gate admits after reading the record, so card 2's `requestContext`, which extends any call with `held` above zero, would move every GitHub call's deadline and detach it from its caller's cancellation through `context.WithoutCancel`.
The sentence "for a call no live entry covers, `held` is zero" states the wanted outcome but contradicts the definition, and the SetGateSlug row cannot tell them apart, because the card's fake `Now` advances only in the gate's `Sleep` (line 22), which an uncovered call never reaches.
Round 1's F1 raised this, and its fixer report says the definition and the row were changed, but the current card carries the round-1 text.

### [MEDIUM] Card 5's conditioned notice is still a .notice file

**Location:** `_lyx/plan/05-orch-conditional-notice.md:28`

**Issue:** The card keeps the `.notice` suffix so that "a watcher built before this change delivers it as a plain notice".
Such a watcher types the revive line with no state check, which breaks D5's bound that the orch gets the line only when its state reads idle-stalled with cause `api-error` at the notice's delivery, and the card states no bound for it.
An old watcher binary is exactly the case D8's re-exec has not yet replaced.
Round 1's F2 raised this, and its fixer report says the `.cnotice` suffix and the `noticeName`, `noticeNames` and `QueueNotice` targets were added, but the current card has neither.
`noticeName` today takes only `now` (`internal/orchengine/notice.go:31`), so carrying the condition in the name re-signs it, and `QueueNotice` (`notice.go:43`) is its caller.

### [LOW] Card 12's error-prefix claim is false

**Location:** `_lyx/plan/12-tender-rename.md:38`

**Issue:** The card says "Error texts keep their `pause:` prefix", but `internal/pauseengine/pause.go:25` and `:27` read `pauseengine: the pause entry is no longer live` and `pauseengine: another live pause entry owns the session`.
Its surgical edits leave those texts, so its own closing check, "A grep for `pausecli` and `pauseengine` outside `_lyx/` and `docs/research/` finds nothing after this card" (line 50), fails.
Round 1's F4 raised this, and its fixer report says it was fixed, but the current card carries the round-1 text.

### [LOW] Card 11 leaves attach's Long stale

**Location:** `_lyx/plan/11-reed-attach-exec.md:34`

**Issue:** `internal/reedcli/attach.go:34-35`'s `Long` says attach "hands the operator's own stdio over to a tmux attach-session child".
After card 11 that is false on Unix, and the card updates only `reedcli/doc.go`.
Round 1's F5 raised this, and its fixer report says it was fixed, but the current card does not mention the `Long`.

### [LOW] Card 10 documents the caller refusal twice

**Location:** `_lyx/plan/10-tender-revive.md:84`

**Issue:** The card adds a "revive: agent outside its own pair" row to `contracts/specs/refusal-spec.md` and also makes the existing caller row ("pause or resume: agent outside its own pair", `refusal-spec.md:439`) name revive.
Both describe the one `refuseCaller` refusal.
Round 1's F6 raised this, and its fixer report says the new row was dropped, but the current card still adds it.

## Focus verdicts

- Card 3, production timeouts: not a defect.
  The page-loop shape and the gate wiring are as the focus asks; the timing margin is F2.
- Card 4, `retryLanded`: not a defect.
  `typeVerified` excludes the `retryLanded` check, which stays in `sendVerified` (`internal/shuttleengine/run.go:1326-1340`), so `SendConditioned` never reads `pending-send.json`.
  The "pending send on the paste path" row runs over `pastePickerEngine`, a `TextPaster` that is also a `SessionCycler` and `InputBoxReader`, the path production's engine takes, with a pending record that would make `retryLanded` report landed, so the row fails if the shortcut is taken.
- Card 5, state-dependence comments: not a defect.
  The card rewords all four sites in `orchengine` that say no watcher decision reads the state: `Session.SessionState`'s comment (`watcher.go:42`), `logIdleStateDisagreement`'s doc (`watcher.go:603`), and `doc.go:151` and `doc.go:154`.
  No other package or doc says this of the orch watcher.
- Card 6, reachability of the safe-restart branches: the phase and live-entry branches are reachable through `Run`, and the state re-read row's `onAlive` input reaches its branch, since a quiet idle tick saves nothing after `StrandAlive`.
  The gaps are the record-read row in F3 and the phase entry time in F4.

## Prior rounds

Read after this review was first saved.
Round 2's six fixes and its gate fix are present in the current cards.
Round 1's six fixes are not: cards 1, 5, 10, 11 and 12 and the round-1 rows of cards 4, 5 and 6 carry the text round 1 reviewed, so F3, F5, F6, F7, F8 and F9 re-raise them.
No prior fixer report disputed or deferred a finding.
