---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/01-pauserecord-network-held.md:16
    summary: Card 1 defines held as the time from entry until the wait returned, which is nonzero for every gated call under a real clock, so card 2 would treat every call in a hub as held, and the fake-clock rows cannot catch it
    fix: In card 1's Intent, define held as zero unless the wait slept at least once, and say how Network tells, e.g. by wrapping the gate's Sleep seam to note that it ran; measure held from entry to the wait's return only in that case. In TestNetwork, make the uncovered SetGateSlug row's gate Now advance on every call (not only in Sleep), and keep its assertion that held is zero, so the row fails an implementation that measures elapsed time unconditionally.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-orch-conditional-notice.md:28
    summary: Card 5 keeps the .notice suffix on purpose, so a watcher built before this change types a conditioned revive notice as a plain one, with no state check; that breaks D5's at-delivery bound and is stated with no bound, and the old binary has no re-exec to end it
    fix: In card 5, give a conditioned notice a file name an old watcher's noticeNames skips, e.g. the condition tag replacing the .notice suffix (<time>-<rand>.when-<state>+<cause>.cnotice), and have noticeNames list both suffixes in arrival order, so only a watcher that reads the condition ever delivers it. Add internal/orchengine#noticeName and internal/orchengine#noticeNames to card 5's Edit list (checked with validate-plan --overlay; no new finding). Replace the sentence saying an old watcher delivers it as a plain notice with one saying an old watcher leaves it queued until a watcher of this build delivers or drops it, and add a TestQueueConditionalNotice assertion that the conditioned file's name does not end in .notice.
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/04-shuttle-conditioned-send.md:39
    summary: Three guard branches the cards prescribe have no test row that reaches them, a gap that recurs across cards 4, 5 and 6
    fix: Add one row per branch to the test that card already plans. TestRunner_SendConditioned in card 4 gets a row for a pane showing no input box, which refuses with a SessionBusyError, types nothing and leaves the clock where it was. TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads in card 5 gets a row where fakeSession.sessionStateErr is set, so the tick returns the error, types nothing and keeps the notice queued. TestRun_ChecksTheBinaryOnlyAtARestartSafeTick in card 6 gets a row with an unreadable pause record at the told RecordPath, e.g. a directory in its place, so the check never runs.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/12-tender-rename.md:45
    summary: Card 12 says a grep for pauseengine finds nothing after the rename, but its surgical edits leave the pauseengine error prefixes of errEntryGone and errOwnedElsewhere in pause.go
    fix: In card 12's list of surgical edits, after the tendercli log-message bullet, add a bullet saying the tenderengine error texts in pause.go that start with pauseengine (errEntryGone, errOwnedElsewhere) now start with tenderengine, and that a grep for the old prefix in tests found no assertion on them.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/11-reed-attach-exec.md:34
    summary: Card 11 updates reedcli/doc.go but leaves attachCmd's Long, which says attach hands stdio to a tmux attach-session child, false on Unix
    fix: In card 11's Intent, beside the reedcli/doc.go sentence, add that attachCmd's Long says attach replaces itself with tmux attach-session on Unix and runs it as a child on Windows, keeping the rest of the Long.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/plan/10-tender-revive.md:84
    summary: Card 10 both adds a revive agent-outside-its-own-pair row to refusal-spec.md and makes the existing caller row name revive, so one refuseCaller refusal is documented twice
    fix: In card 10's refusal-spec sentence, drop the new row revive agent outside its own pair and keep only the edit that makes the existing caller row (pause or resume, agent outside its own pair) name revive in its title, trigger and way forward; keep the new revive unknown-slug row.
---

### [MEDIUM] `held` measures elapsed time on every gated call

**Location:** `_lyx/plan/01-pauserecord-network-held.md:16`

**Issue:** Card 1 says `Network` returns `held`, "the time from its entry until the wait returned", and also that `held` is zero for a call no live entry covers.
These two disagree once a real clock runs.
`WaitUncovered` (`internal/pauserecord/hold.go`) always reads the record before it returns, so the elapsed time is a few microseconds above zero even when nothing covers the call.
`WaitUncovered` returns only an error, so nothing tells `Network` whether it slept, and the card doesn't say how `Network` would know.
Card 2 sends every call with `held` above zero down the `context.WithoutCancel` path.
An implementation that follows the first sentence therefore moves the deadline of every gated, unheld call, which is every call in a hub, and ignores a late cancellation on all of them.
That breaks D6's bound that "an unheld request keeps exactly its deadlines", which card 2's ImpactSummary repeats as "bit for bit".
The planned rows can't catch it.
Card 1's fake `Now` advances only inside `Sleep`, so an unconditional measurement still reads zero there.
Card 2's unheld row only checks that the call fails with `DeadlineExceeded` at about 50 ms, which a few-microsecond extension still does.

### [MEDIUM] An old watcher types a conditioned notice with no state check

**Location:** `_lyx/plan/05-orch-conditional-notice.md:28`

**Issue:** The design puts the condition in the file name but keeps the `.notice` suffix "so a watcher built before this change delivers it as a plain notice".
`noticeNames` (`internal/orchengine/notice.go`) keeps every name ending in `.notice`, so a watcher from before this change delivers a revive notice as soon as the orch reads idle, even when the orch has recovered.
D5 bounds revive to typing only into an orch whose state reads idle-stalled with cause `api-error` at the notice's delivery, and card 10 repeats that bound.
A watcher started before the deploy runs a binary with no card-6 re-exec, so it keeps running until something restarts it.
The card turns this bypass on deliberately and names no bound for it.
The attack-surface rule needs that bound stated, and D5 needs the bypass closed.

### [LOW] Three prescribed guard branches have no test row

**Location:** `_lyx/plan/04-shuttle-conditioned-send.md:39`

**Issue:** Card 4 says the conditioned reading refuses "a pane too short or showing no input box".
None of the rows in `TestRunner_SendConditioned` reaches that refusal.
Card 5 (`05-orch-conditional-notice.md:33`) says "a state read error returns the error and keeps the notice".
No row of `TestWatcher_ConditionalNoticeDeliveredOnlyWhileItsStateReads` sets `sessionStateErr`, though `fakeSession` already has that field.
Card 6 (`06-orch-watch-binary-reexec.md:28`) says "a state or record read error is not safe".
No row of `TestRun_ChecksTheBinaryOnlyAtARestartSafeTick` feeds it an unreadable state or record.
The same omission recurs in each card's test list, so it is filed once here.

### [LOW] Card 12's grep claim misses the engine's error prefixes

**Location:** `_lyx/plan/12-tender-rename.md:45`

**Issue:** `internal/pauseengine/pause.go` declares `errEntryGone` and `errOwnedElsewhere` with texts that start `pauseengine:`.
Card 12's surgical edits cover package clauses, imports and the `pausecli:` log messages, but not these texts.
Its closing claim that a grep for `pauseengine` finds nothing then fails.

### [LOW] `attach`'s help text goes stale on Unix

**Location:** `_lyx/plan/11-reed-attach-exec.md:34`

**Issue:** `attachCmd`'s `Long` in `internal/reedcli/attach.go` says attach "hands the operator's own stdio over to a tmux attach-session child".
After card 11 that is false on Unix, where the process becomes tmux.
The card edits `attachCmd` and updates `reedcli/doc.go`, but leaves the help text that operators read as it is.

### [LOW] One refusal gets two refusal-spec rows

**Location:** `_lyx/plan/10-tender-revive.md:84`

**Issue:** Card 10 routes revive's caller refusal through the existing `refuseCaller`, with one message for all three verbs.
It still asks for a new "revive: agent outside its own pair" row and also for the existing caller row (`contracts/specs/refusal-spec.md:439`) to name revive.
That documents the same refusal twice.

## Rejected

- Card order (focus item 2): cards 9 and 10 build only on cards 4, 5 and 7, card 3's test builds on card 2, and card 11's shortcut rule builds on card 8's shortcut file. No card depends on a later one. Not a defect.
- Edit glyphs and re-sign arrows (focus item 5): every glyph on cards 1–6 and 8–11 resolves in the tree (checked by grep), and the one changed signature, `pauserecord#Network`, carries its arrow. Card 7 lists `proc_linux.go` and `proc_windows.go` as files under Edit because they receive the new `ListProcesses` declarations its Create names, not changes to existing symbols. Not a defect.
- Test economy (focus item 6): card 2's new test covers the transport's deadlines, which no existing test reaches, and it reuses the existing classification and re-step tests by name instead of adding rows. Card 3 adds a row to `TestListOpenIssues_PaginatesAndExcludesPullRequests`, and card 9 adds subtests to `TestStatus`. Card 10's `TestRevive` pins a new exported function. Not a defect.
- Verify coverage (focus item 7): the `## verify:` section matches the template's order exactly. The tmux-tier `TestSmokeLifecycle` change in card 11 is compiled by `go vet -tags tmux`, which the rubric exempts. Not a defect.
- CLI shape (focus item 8): `tender` is a non-invocable group with no audience, and each leaf carries the operator audience, which by `clihelp` covers the orch acting for the operator. That is what the audience bullet requires. Not a defect.
- Fidelity (focus item 1): D1–D9 and every Constraint are carried by some card, and every card's scope traces to a decision. Card 3's `fetchOpenPRHeads` and `CommentAndClose` are licensed by D6's several-request clause. The record's carry-over block is not part of the answer key.
- Attack surface (focus item 3): cards 4, 6 and 11 state what the change lets through and its bound. Card 5's gap is F2.
- Reachability (focus item 4): card 4's state, draft, stray, activity, picker and not-landed refusals each have a row. Card 5's drop-at-delivery has its busy and idle-done rows. Card 6's restart-safe gating has its phase and hub-entry rows. The read-error and no-input-box gaps are F3.
