---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-orch-watch-binary-reexec.md:47
    summary: Card 6's non-idle row saves PhaseHandoffRequested with no PhaseEnteredAt, so the first tick's handoff timeout returns the phase to idle and the check runs, which never reaches restartSafe's non-idle branch
    fix: In card 6's Intent, reword the row "a non-idle phase, `PhaseHandoffRequested`, saved before `Run`" to say the state is saved before `Run` with `Strand` s1, `Phase` `PhaseHandoffRequested` and `PhaseEnteredAt` set to the env clock's now, so `tickHandoff`'s `HandoffTimeout` check (`now.Sub(started) >= w.cfg.HandoffTimeout()`) does not return it to idle while the row runs, and add that the row also asserts the persisted phase still reads `PhaseHandoffRequested` after `Run` returns, so the row fails if the phase left the handoff before the check was asked.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/03-github-per-request-deadlines.md:31
    summary: Card 3's held-page row passes with only 30 ms of margin on page 1, since page 1 answers after 170 ms against a 200 ms per-request budget, a wall-clock margin a loaded gate can eat
    fix: In card 3's Intent, change the held-page row's numbers to a 400 ms `listOpenIssues` budget, a first page answered after 300 ms (so a shared deadline has 100 ms left when page 2 starts), the gate's `Sleep` of 150 ms, and a page 2 answered after 200 ms; update the sentences that state the margins to say a fresh per-page deadline leaves page 2 its whole 400 ms after release, 200 ms of margin, and a shared deadline leaves it its 100 ms remainder, so the call fails with a deadline error.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-orch-conditional-notice.md:28
    summary: Card 5 keeps the .notice suffix on a conditioned notice on purpose, so a watcher still running a pre-change binary types a queued revive notice with no state check, which breaks D5's at-delivery drop with no stated bound; round 1's fix for this is no longer on the card
    fix: "In card 5, add `internal/orchengine#noticeName` -> `func noticeName(now time.Time, when *NoticeCondition) (string, error)` after `internal/orchengine#Notice` on the Edit list, then `internal/orchengine#QueueNotice`, and `internal/orchengine#noticeNames` after `internal/orchengine#ListNotices`. Replace Intent lines 28-30 with: a conditioned notice is named `<time>-<rand>.when-<state>+<cause>.cnotice`, which `noticeName` builds when told a condition and `QueueNotice` calls with nil; an old watcher's `noticeNames` skips that suffix, so a watcher built before this change leaves it queued until a watcher of this build delivers or drops it; `noticeNames` lists both suffixes in arrival order, and `ListNotices` parses the tag into `Notice.Condition`. In `TestQueueConditionalNotice`'s first row, add that the conditioned file's name does not end in `.notice`. `lyx loom validate-plan --overlay` over this card passed with no finding beyond the baseline's."
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-pauserecord-network-held.md:16
    summary: Card 1 defines held as the time from entry until the wait returned, which is above zero for every gated call on a real clock, while the same paragraph says held is zero for a call no entry covers, so card 2 would extend every call; round 1's fix for this is no longer on the card
    fix: "In card 1's Intent, replace the sentence defining `held` with: `held` is zero unless the wait slept at least once; `Network` tells by wrapping the `Sleep` it hands the `Waiter` in a closure that notes it ran, and only then measures `held` from entry to the wait's return. In the `TestNetwork` paragraph, say the SetGateSlug row's gate gets a fake `Now` that advances on every call, not only in `Sleep`, and its uncovered first call keeps asserting `held` zero, so the row fails an implementation that measures elapsed time unconditionally. No target changes."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/04-shuttle-conditioned-send.md:39
    summary: Two guard branches the cards prescribe have no row that reaches them, card 4's pane showing no input box and card 5's state read error at delivery; round 1's rows for both are no longer on the cards
    fix: "In card 4's `TestRunner_SendConditioned` rows, add: no input box, where the capture has no caret, gives a `*SessionBusyError`, types nothing and leaves the clock unmoved. In card 5's `TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads` rows, add: `sessionStateErr` set, so the tick returns the error, types nothing and keeps the notice queued. Both are rows in tests the cards already plan, so no target or budget changes."
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/12-tender-rename.md:45
    summary: Card 12 says a grep for pauseengine finds nothing after the rename, but its surgical edits keep the pauseengine prefix in the error texts of errEntryGone and errOwnedElsewhere in pause.go
    fix: "In card 12's list of surgical edits, after the bullet on `tendercli`'s log messages, add a bullet: `tenderengine`'s error texts in `pause.go` that start with `pauseengine:` (`errEntryGone`, `errOwnedElsewhere`) now start with `tenderengine:`, and no test asserts on the old prefix. No target changes: `internal/pauseengine/pause.go` is already a Rename pair on the card."
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/11-reed-attach-exec.md:19
    summary: Card 11 makes attach exec tmux on Unix but leaves attachCmd's Long text, which says attach hands stdio to a tmux attach-session child, unchanged
    fix: "In card 11's Intent, after the `handOver` paragraph, add: `attachCmd`'s `Long` says attach execs tmux in place on Unix and hands stdio to a tmux child on Windows, and keeps the rest of its text. No target changes: `internal/reedcli#reedCLI.attachCmd` is already on the Edit list."
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/plan/10-tender-revive.md:84
    summary: Card 10 adds a refusal-spec row for revive by an agent outside its own pair and also has the existing caller row name revive, so one refusal is documented twice
    fix: "In card 10's Intent, change the `refusal-spec.md` sentence to: the pause section gains the row \"revive: unknown slug\" with the way forward its refusal carries, and the existing row \"pause or resume: agent outside its own pair\" names revive in its title, trigger and way forward. No target changes."
---

### [MEDIUM] Card 6's non-idle row never reaches the non-idle branch

**Location:** `_lyx/plan/06-orch-watch-binary-reexec.md:47`

**Issue:**
The row "a non-idle phase, `PhaseHandoffRequested`, saved before `Run`: the check never runs" names only the phase.
`newWatchEnv` (`internal/orchengine/watcher_test.go:231`) runs on a fake clock fixed at 2026-01-02 with `HandoffTimeoutS: 100`.
`Watcher.tick` (`internal/orchengine/watcher.go:211`) loads the state, and `tickHandoff` (`internal/orchengine/watcher.go:796`) computes `started := st.PhaseEnteredAt` and returns to idle through `toIdle(st, "handoff timed out")` when `now.Sub(started) >= w.cfg.HandoffTimeout()`.
A state saved with a zero `PhaseEnteredAt` therefore leaves the handoff on the first tick, and `restartSafe`'s re-read then sees phase idle, so the check runs.
The named input gets past the tick's own `LoadState`, but the tick's handoff timeout turns it into an idle state before `restartSafe` reads it.
As written, the row either fails against a correct implementation or gets patched at implementation time with no plan guidance.
The rubric's reachability bullet covers this case: a named input that an upstream check removes before it reaches the prescribed branch.
The card's other `restartSafe` branches do reach.
The state-unreadable row's input passes the tick's `LoadState` before the `onAlive` hook moves the file: `LoadState` runs at `watcher.go:212`, before `StrandAlive` at `watcher.go:220`.
After the second tick's `initCursor` nothing saves, because the color is typed on the first tick and `probeIdle` saves only while `StuckByHold` is set.
In the record-unreadable row, `pauseTick` (`internal/orchengine/pause.go:41`) tolerates a `pauserecord.Read` error, which `pauserecord.Read` (`internal/pauserecord/record.go:84`) returns for a directory.
In the live-hub-entry row, the tick returns held with a nil error, and `restartSafe`'s record read then sees the entry.

### [LOW] Card 3's held-page row leaves 30 ms of wall-clock margin on page 1

**Location:** `_lyx/plan/03-github-per-request-deadlines.md:31`

**Issue:**
The row gives `listOpenIssues` a 200 ms per-request budget and has the handler answer page 1 after 170 ms, so the passing case depends on page 1 finishing, go-github decode included, within 30 ms of real time.
The same 30 ms also separates the shared-deadline remainder from page 2's 60 ms answer, so the margin cannot be raised without moving all the numbers together.
Under a gate running other packages in parallel, 30 ms of scheduling delay can fail a correct implementation.
The testing rules treat a test that passes most of the time as a bug.
Scaling the numbers keeps the row discriminating: the shared remainder stays below page 2's answer time, and page 2's answer time stays below the fresh budget.
The rest of focus item 1 is not a defect.
`callTimeout` stays a `const`, the test's budget reaches the code as the `listOpenIssues` `perRequest` parameter, card 2's client timeout is the `authRT.timeout` field, and card 1's clock is the `Gate.Now` field, so no production timeout a test shortens is a package-level `var`.
The row builds its client with `githubclient.NewWithBaseURL`, which wraps the transport in `authRT` (`internal/githubclient/githubclient.go:30`), so the pause gate holds page 2.
`resolveToken` takes `GH_TOKEN` before the cache (`internal/githubclient/token.go:88`), so the row touches no credential cache.
Round 2's F6 widened this margin from 20 ms to 30 ms; this finding asks for more, and disputes nothing.

### [MEDIUM] Card 5 lets a pre-change watcher type a conditioned notice unchecked

**Location:** `_lyx/plan/05-orch-conditional-notice.md:28`

**Issue:**
Card 5 keeps `.notice` as the file's last suffix and says, as intended behavior, that "a watcher built before this change delivers it as a plain notice".
`noticeNames` (`internal/orchengine/notice.go:114`) keeps every file whose name ends in `noticeSuffix`, so an old watcher lists the conditioned file and types its line with no state check.
After a deploy, `lyx tender revive` runs the new binary while a long-lived `lyx orch watch` can still run the old image, and only the new image carries card 6's re-exec.
D5's bound, that the watcher drops a queued revive notice at delivery once the orch no longer reads idle-stalled with cause `api-error`, then does not hold, and the card states no bound for the gap.
Round 1 raised this as its F2, and its fixer report says the card moved to a `.cnotice` suffix with `noticeName`, `noticeNames` and `QueueNotice` on the Edit list.
The card as it stands carries none of that, so the fix regressed.

### [MEDIUM] Card 1's definition of `held` contradicts its own zero case

**Location:** `_lyx/plan/01-pauserecord-network-held.md:16`

**Issue:**
Card 1 defines `held` as "the time from its entry until the wait returned, measured on the gate's clock", and the next sentence says `held` is zero for a call no live entry covers.
`Network` (`internal/pauserecord/hold.go:202`) calls `WaitUncovered`, which returns only an error, so `Network` has no reading of whether it slept.
On a real clock, the entry-to-return time includes the record read, so it is above zero for every gated call.
Card 2's `requestContext` then takes its held branch for every request: `context.WithoutCancel` with the caller's deadline moved by that time.
That breaks card 2's claim that an unheld call keeps exactly its deadlines.
Round 1's F1 raised this, and its fixer report says the card measured `held` only when the wait slept and gave the SetGateSlug row a clock that advances on every call; neither is on the card now.

### [LOW] Two prescribed guard branches have no reaching row

**Location:** `_lyx/plan/04-shuttle-conditioned-send.md:39`

**Issue:**
Card 4 prescribes a `*SessionBusyError` refusal for "a pane too short or showing no input box", but `TestRunner_SendConditioned` has no row for either.
Card 5 prescribes "a state read error returns the error and keeps the notice" in `deliverNotice`, but no row of `TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads` sets `fakeSession.sessionStateErr`.
Round 1's F3 added both rows and the card 6 record row, and only the card 6 row survives.
This is one gap that recurs across cards, so it is filed once.

### [LOW] Card 12's grep claim misses two error prefixes

**Location:** `_lyx/plan/12-tender-rename.md:45`

**Issue:**
`internal/pauseengine/pause.go:25` and `:27` declare `errEntryGone` and `errOwnedElsewhere` with texts starting `pauseengine:`.
Card 12's surgical edits change only package clauses, imports, qualifiers and `tendercli`'s log messages, and say error texts keep their `pause:` prefix, which these two do not carry.
The card's closing claim, that a grep for `pauseengine` outside `_lyx/` and `docs/research/` finds nothing, then fails.
Round 1's F4 fix for this is no longer on the card.

### [LOW] Card 11 leaves attach's help text describing a child

**Location:** `_lyx/plan/11-reed-attach-exec.md:19`

**Issue:**
`attachCmd`'s `Long` (`internal/reedcli/attach.go:34`) says attach "hands the operator's own stdio over to a tmux attach-session child".
On Unix, card 11 replaces the process with tmux, so that text goes stale, and the card's only doc change is `reedcli/doc.go`.
Round 1's F5 fix for this is no longer on the card.

### [LOW] Card 10 documents the revive caller refusal twice

**Location:** `_lyx/plan/10-tender-revive.md:84`

**Issue:**
Card 10 adds the row "revive: agent outside its own pair" and also has the existing caller row, "pause or resume: agent outside its own pair" (`contracts/specs/refusal-spec.md:439`), name revive.
`refuseCaller` produces one refusal for all three verbs, so two rows would document it twice.
Round 1's F6 fix, which dropped the new row, is no longer on the card.

## Focus verdicts

- Card 3: see F2. The timeouts are parameters, fields or consts, and the held-page client goes through the `githubclient` auth transport.
- Card 4: not a defect. In today's `sendVerified` (`internal/shuttleengine/run.go:1326`), `retryLanded` sits in the paste branch before `pasteVerified`, and neither `pasteVerified` nor `confirmPasted` reads `pending-send.json`, so a `typeVerified` cut that holds only `pasteVerified` and the typing path cannot consult the record. The "pending record ignored" row reaches the paste path: `pickerEngine` embeds `readinessEngine` and `idleEngine`, so it is an `InputBoxReader` and a `SessionCycler`, and the row adds `TextPaster`. Its record's offset before `START` makes `retryLanded` read as landed, so the row fails an implementation that consults it.
- Card 6: see F1. The state-unreadable, record-unreadable and live-hub-entry rows reach their branches.
- Card 5: not a defect. `internal/orchengine`'s claims that the state decides nothing are the `Session.SessionState` comment (`watcher.go:42`), `logIdleStateDisagreement`'s doc (`watcher.go:603`) and `doc.go:151` and `doc.go:154`, and card 5 lists each of them.
