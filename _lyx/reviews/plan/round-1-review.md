---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: '_lyx/plan/10-orch-note-gate-no-reply.md, Tests bullet on TestWatcher_NoteRequestRendersTemplateFileBeforeTyping'
    summary: 'Card 10 plans a new assertion that the hard note request names no --reply, a pin on a stencil literal, which PATTERN-test-economy forbids (no test pins a stencil static prose)'
    fix: 'In card 10, rewrite the TestWatcher_NoteRequestRendersTemplateFileBeforeTyping bullet so the test drops every assertion on stencil literals (the written-path text, lyx creel post --reply, [creel]) and plans no new one: drop the planned assertion that the hard request names no --reply. Keep only the Go-filled values (the note template path and PendingHandoff path in both requests) and the existing hard-versus-soft family rule that the hard request offers no defer. Review, not a test, guards that the reworded stencils ask for no written reply.'
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: '_lyx/plan/07-creel-delivery-teardown.md, Intent paragraph on the Warn line and the TestHubDelivery_SkipsAWorktreeThatNoLongerResolves row with the .git replaced by an empty directory'
    summary: 'The Warn skip names git worktree repair from the prime as the way forward, but repair refuses a worktree whose .git is a directory, the very Warn case the card test pins, so the named step cannot end the skip'
    fix: 'In card 7, name git worktree repair <path> as the way forward only when the worktree .git is a file (a broken gitfile link); when .git is a directory, the Warn line names the cause (.git is a directory, not a worktree link) and the way forward of removing and re-creating that worktree through the hub teardown, or names none. Update the empty-.git-directory test row to assert the line that case logs, and add the gitfile case to the Warn rows if it is to keep the repair wording.'
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: '_lyx/plan/11-orch-compact-withdraw-queued.md, Intent sentence on withdrawQueuedCompact and a ClearResumeNoticed failure'
    summary: 'On a ClearResumeNoticed failure card 11 keeps ResumeLineWithdrawn set and saves it, but nothing on the compacting path retries the clear, so the withdrawn resume line is never posted again'
    fix: 'In card 11, state that a ClearResumeNoticed failure in withdrawQueuedCompact fails the tick and leaves the phase compacting with the boundary unconsumed, and that the next qualifying tick re-runs ClearResumeNoticed from State.ResumeLineWithdrawn before completing, even when the /compact outcome then reads withdrawn (the withdraw precondition no longer holds). Extend the unwritable-pause-record test row so the following tick, with the record writable again, clears the entry ResumeNoticed and completes the phase.'
  - id: F4
    severity: MEDIUM
    class: scope
    origin: handler
    location: '_lyx/plan/05-reed-pane-elsewhere.md, Edit list; internal/reedcli/status.go:45'
    summary: 'D2 says the strand status row names the placement, but lyx reed status builds its own row map (guid, name, paneId, live, retiring) and no card adds StrandStatus.Elsewhere to it, so the operator sees the placement only for the orch strand'
    fix: 'Add internal/reedcli#reedCLI.statusCmd to card 5 Edit list (validated with lyx loom validate-plan --overlay together with F6), and state in its Intent that each row gains an elsewhere key carrying StrandStatus.Elsewhere, empty when none, that the verb Long help names it, and that internal/reedcli/doc.go is updated if it lists the row keys.'
  - id: F5
    severity: LOW
    class: design
    origin: lens:guard-reach
    location: '_lyx/plan/11-orch-compact-withdraw-queued.md, Intent on the no-id branch; internal/orchengine/watcher.go:350'
    summary: 'The no-id branch of the withdraw precondition is reached only after a creel failure in the same tick, because recoverPending runs before tickCompacting and resends or adopts the step first; no test row reaches it and the Intent does not name that ordering'
    fix: 'In card 11, state that tick runs recoverPending before tickCompacting, so a compacting step with no id has already been adopted or re-sent before the boundary check, and that the no-id branch covers only a creel failure in that same tick; add a row to TestCompact_BoundaryWithdrawsAStillQueuedCompact where the fake creel Sent and Inject fail in the boundary tick, leaving Pending with no id, and assert the withdraw runs.'
  - id: F6
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: '_lyx/plan/03-reed-engine-liveness.md, Edit list; internal/shuttleengine/stop.go:84'
    summary: 'strandRemovable doc comment says an own-window strand is removable when its pane is present but dead; under D1 a present, not-dead shell-only pane also reads not live, and no card rewords the comment cards 3 and 4 rely on'
    fix: 'Add internal/shuttleengine#strandRemovable to card 3 Edit list (validated with lyx loom validate-plan --overlay together with F4) and state that only its doc comment changes, to an own-window strand whose pane is present but not live, a dead pane or a shell-only one.'
  - id: F7
    severity: LOW
    class: consistency
    origin: handler
    location: '_lyx/plan/09-orch-request-pause-discount.md, Intent on Watcher.unpausedSince; internal/pauserecord/sleep.go:51'
    summary: 'Card 9 re-derives the elapsed time less the excluded hub intervals, which pauserecord.AwakeSince already computes, floored at zero'
    fix: 'In card 9, have Watcher.unpausedSince read the record and return pauserecord.AwakeSince(from, now, coverage.Intervals) with the CoverageOf("", now) intervals, instead of restating now.Sub(from) less pauserecord.Excluded.'
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: '_lyx/plan/04-conflict-stop-shell-only.md, test row with Live false, OwnWindow false, Present true'
    summary: 'The planned row pins a state that cannot occur, since D1 condition 2 establishes that every strand bound to a present pane is own-window'
    fix: 'In card 4, drop the Live false, OwnWindow false, Present true row from TestConflictSessionStopper, keeping the stopped own-window shell-only row and the no-present-pane row.'
  - id: F9
    severity: NIT
    class: design
    origin: lens:guard-reach
    location: '_lyx/plan/03-reed-engine-liveness.md (nil paneAge fallback) and _lyx/plan/12-orch-withdraw-and-request-status.md (hubPauseHolds empty path)'
    summary: 'Two planned branches guard states the code wiring rules out: a nil Engine.paneAge, which New always sets, and an empty pauseRecordPath, which PersistentPreRunE always sets or fails on'
    fix: 'In card 3, drop the nil-paneAge fallback and its TestEngineAliveIDs row, or state the hand-built test Engine that needs it; in card 12, drop the empty-path branch of hubPauseHolds, keeping only the unreadable-record answer of false.'
---

# Plan review, round 1

The plan carries every Decision D1 to D7 and every Constraint of the decision record.
D1 is carried by cards 1, 2, 3 and 4, D2 by cards 5 and 13, D3 by card 13, D4 by card 10, D5 by cards 8, 9 and 12, D6 by card 6, and D7 by cards 7 and 11.
The claude.yaml `binary` template comment is on card 3, and the hub rules removal stays out of every card, as the record's Out list says.
The card order is sound.
Cards 3, 5, 6, 12 and 13 use only symbols that earlier cards create: `paneProcessAge`, `engineAliveIDSet`, `StrandStatus.Elsewhere` and `WithdrawCycleRequest`.
No card carries a `**Verify:**` line.
The `Edit` cards list member glyphs, and the two re-signed functions, `directoryRows` and `planReconcile`, carry the arrow.
The attack-surface statements on cards 6, 12 and 13 name what each can now let through and its bound.
`lyx loom validate-plan --overlay` with `{}` answered ok.
The combined overlay of the F4 and F6 target additions also answered ok.

### [BLOCKING] Card 10 plans a static-prose assertion on the hard note request

**Location:** `_lyx/plan/10-orch-note-gate-no-reply.md`, the Tests bullet on `TestWatcher_NoteRequestRendersTemplateFileBeforeTyping`.

**Issue:** The card plans to assert that the hard request "names no `--reply`".
`--reply` is a literal in `contracts/stencils/orch/orch-template-handoff.md`, not a value Go fills, so the assertion pins static prose.
`PATTERN-test-economy` forbids that.
Go's choice between the hard and soft stencils is already pinned by the existing check that the hard request offers no `defer`.

### [MEDIUM] Card 7's Warn way forward cannot repair the case its test pins

**Location:** `_lyx/plan/07-creel-delivery-teardown.md`, the Warn-line sentence and the empty-`.git`-directory row.

**Issue:** I ran `git worktree repair` on a scratch repository whose linked worktree's `.git` was replaced by an empty directory.
From the main worktree it answered `error: .git is not a file` with exit 1, and with the worktree path given it answered `unable to locate repository; .git is not a file`.
So the step the Warn line names leaves the worktree skipped in exactly the case the card's test drives.

### [MEDIUM] Card 11 never retries a failed resume-noticed clear

**Location:** `_lyx/plan/11-orch-compact-withdraw-queued.md`, the `withdrawQueuedCompact` clear-failure sentence.

**Issue:** `State.ResumeLineWithdrawn` is re-read only by `retryFreshLaunch`, which runs only with `FreshLaunchFailure` set (`internal/orchengine/watcher.go:418`), and by `resetForOtherStrand`, which runs only on a strand change (`watcher.go:852`).
After the withdraw, the inject's outcome reads withdrawn, so card 11's own precondition never re-runs `withdrawQueuedCompact`.
The phase completes, and the pause entry keeps `ResumeNoticed` set, so the resume line the withdraw removed is never posted again.
D7 asks for this clear to work "as the watcher's other withdraw sites do", and those sites retry.

### [MEDIUM] `lyx reed status` does not show the placement D2 adds

**Location:** `_lyx/plan/05-reed-pane-elsewhere.md`; `internal/reedcli/status.go:45`.

**Issue:** D2 says "its status row names the placement (`session:window`)", and its rationale is that "the operator learns where a pane they moved out went".
Card 5 adds `StrandStatus.Elsewhere`, but reed's `status` verb renders a fixed map of guid, name, paneId, live and retiring.
So the placement reaches the operator only through `lyx orch status`, and only for the orch strand.

### [LOW] Card 11's no-id branch is reached only after a same-tick creel failure

**Location:** `_lyx/plan/11-orch-compact-withdraw-queued.md`; `internal/orchengine/watcher.go:350`.

**Issue:** `tick` runs `recoverPending` for any non-idle `Pending` with no id before the phase switch, adopting or re-sending the step.
So by the time `tickCompacting` checks for the boundary, an id is normally set.
The "pending step has no id yet" disjunct holds only when `Sent` or `Inject` failed in that tick, and no planned row drives it.

### [LOW] `strandRemovable`'s doc comment goes stale under D1

**Location:** `internal/shuttleengine/stop.go:84`.

**Issue:** The comment reads "an own-window strand whose pane is present but dead".
After card 3, a present pane holding a bare shell reads not live without being dead.
The rule in code still removes it, but its documented reason no longer matches.

### [LOW] Card 9 restates `pauserecord.AwakeSince`

**Location:** `_lyx/plan/09-orch-request-pause-discount.md`; `internal/pauserecord/sleep.go:51`.

**Issue:** `AwakeSince(at, now, excluded...)` already returns the span less the union of excluded intervals, floored at zero.
`unpausedSince` re-derives the same thing from `Excluded`.

### [NIT] Card 4 plans a row for a state D1 rules out

**Location:** `_lyx/plan/04-conflict-stop-shell-only.md`.

**Issue:** D1 condition 2 establishes that every bound pane is own-window, since `validateDisplay` admits only `own-window`, `overview` and `hidden`.
A strand with `OwnWindow: false` and a present pane cannot occur.

### [NIT] Two defensive branches guard wiring that cannot be absent

**Location:** cards 3 and 12.

**Issue:** `New` always sets `paneAge`, and `PersistentPreRunE` always sets `pauseRecordPath` or fails the verb, and the card's own `newTestCLI` sets it too.
The code-quality rule forbids a code path for a state the code's own guarantees rule out.

## Rejected

- guard-reach: hub-pause time also counts toward the compacting phase's own timeout. D5 names only the stale-request and handoff clocks, so extending the discount would be scope the answer key does not license.
- guard-reach: card 2's pid-at-or-below-zero and negative-age guards. `/proc` content is OS input read across a boundary, so validating it is allowed.
- handler, considered: a test-economy finding against card 1's new `TestParseWindowPanes`. No existing test calls `parseWindowPanes` directly, and the tail fields are new behavior, so it stays.
