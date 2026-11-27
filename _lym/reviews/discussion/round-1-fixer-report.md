---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed-differently
    reason: "The proposed key, the halt's row and history entry, misses a recurrence that folds into the same entry (internal/shedengine/run.go folds an identical consecutive call and raises only repeats), and the orch's notice carries no history index. D7 gains a `halt:` stamp, the row, history index and repeats, carried on the entry and the notice, and D8 keys the dedup on the slug and that key; a keyless post dedups on the slug alone."
  - id: F17
    disposition: fixed-differently
    reason: "The proposed split for the held notice too would make it fire for an agent holding an unread reply, which internal/shuttleengine/doc.go treats as no hold, since the reply starts a turn in a live session. Only the quiet notice reads the split probe; the held notice keeps the combined `creelengine.Store.AwaitsReply` probe."
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
  - id: F22
    disposition: fixed
  - id: F23
    disposition: fixed
  - id: F24
    disposition: fixed
  - id: F25
    disposition: fixed
---

# Round 1 fixer report

Every change is in `_lym/discussion/decision-record.md` unless named otherwise.

## Changes

- F1: D6 under `unattended` never accepts a round holding an open BLOCKING finding or a red gate, at any cause, and hands it on.
  `arbiterstop.Stop` gains the level and the arbiter's prompt names it.
  D2's hard-cap row says so.
  D6's criterion adds a hard-cap escalation handed on under `unattended`.
- F2: the budget accept is `true` only, and is handed on under `false` and `unattended`.
  D6's Bound states the one change at `false`, the admission and one continue.
  The criterion reads "accepted under `true`, handed on under `false` and `unattended`".
- F3: D12 covers `true` and `unattended`.
  D7 has the orch relay a `settle: operator` halt in its pane while its button is off.
  D7's and D12's criteria cover this.
- F4: D10 chases three stalls: a `settle: orch` halt, an open agent question and a dead driver past its caps.
  D2's dead-driver row gains the operator cell "`unattended`: still dead one interval after D10's chase".
  D10's criterion covers both new cases.
- F5: D2 states that the operator column never applies at `false`, and lym's column applies at every level, D4's rows included.
  D1's `false` line now says what "as today" means.
  D2's criterion names `false` and that no class reaches `operator` there.
- F6: the stale-build row gains "third halt at the row".
  D2 states why `driver_not_parked` and dead driver are no stop at a row and cannot repeat unseen.
  The permission-dialog row is dropped (F15).
- F7: the rows the driver cannot detect are marked *(orch)*, cancel added to the agent-question row.
  D2 says lym stamps none of them, and its claim now reads "No other Go package".
  D7's relay list names exactly those rows, the cancel under `true` included.
- F8: D3 lists the entries that count: a handed-on escalation's Awaiting entry once, a settle record carrying its class and halt index, and a tackler resume from its attempt.
  It excludes an interrupted-reinvoke re-step after a wake and states the match by row and class, not exact stop key.
  D3's criterion adds the four cases.
- F9: D6 reads the segment's `round-<n>-circling-decision.md` files, decision `continue` and cause `budget`, as the spent continue.
  It states that this per-segment count bounds the continue.
  It defines a missing or older fixer report as marking nothing `deferred`, and states that the arbiter's settings allow `lym board upsert` for kind note.
- F10: D4's pause paragraph is rewritten from `internal/dobbyengine/doc.go` and `liftPending` in `internal/dobbyengine/passes.go`, and the hand-back is matched by the pause entry's scope.
  The resume-line sentence and the Notes line are dropped.
  The premise that `queueResumeLine` exists nowhere is false: it is in `internal/orchengine/pause.go:250`.
  It posts `pauserecord.ResumeLine` to the orch's own interrupted session, as the review's summary says, so the record's reading of it was wrong and the change stands.
- F11: a hold's pause entry carries a hold reason and is never resumed by the lift.
  After the self-check it also stands through a `lym tender resume` with no slug, as a bolt-hold entry does (`internal/tendercli/cli.go`), and only a resume naming the slug lifts it.
  D4's criterion covers both.
- F12: both verbs gain a no-wait option, which batten passes on its child start and its resume; D5's criterion adds it.
- F13: the run-lock-held branch on a non-running run waits under the same cap and wake rule; D5's criterion adds it.
- F14: the stamps live on the halt's history entry in the child status, `ChildReading` gains them from the newest entry, and the next transition supersedes them; D10 reads the same entry.
- F15: the dialog row and D9's dialog bullet are dropped.
  Scope, Out cites `validatePermissionMode` in `internal/shuttleengine/claudeengine/command.go`: autonomous sessions skip permissions and refuse the prompt mode.
  The support log's rejected alternative is updated.
- F16: fixed differently; see the frontmatter reason.
  D7 adds the `halt:` stamp, D8 dedups on slug and key, and D10 keys a question and a dead driver.
  D7's, D8's and D10's criteria follow.
- F17: fixed differently; see the frontmatter reason.
  D11 names `creelengine.Store.AwaitsReply` as the probe split for the quiet notice and keeps it whole for the held notice.
  D11's criterion follows.
- F18: Scope, Out names the hung command or gate as section 2's judgment class, with the notice D11 gives the orch.
- F19: every bullet the finding lists carries its tier.
- F20: D7 has the orch file the self-report of a `settle: operator` third halt without relaying it; D7's Review criterion says so.
- F21: the second option: under `unattended` the orch leaves a halt with no settling action open to D10's post.
  After the self-check, D7 states the latency of at most two chase intervals.
- F22: the dead-driver row says it records the dobby's and batten's caps and does not change them.
- F23: D4 states for each new row where its class is raised as a named kind or field; D4's criterion adds that no row matches on message text.
- F24: the Notes line names `AppendSettleRecord` in `internal/sheddriver/settle.go` and the tackler attempts in `tacklerrecords.go`.
- F25: Open risks drops the row name.

## Self-check

- fix-claims, LOW, D5 names `childSpawnError` as the retry site: fixed; D5 names `internal/battenshed/innerrun.go` as the retry.
- fix-claims, LOW, D7 reads as if `ChildReading` already reads entry fields: fixed; D7 says it gains them.
- fix-consistency, MEDIUM, D10's question and dead-driver posts have no halt entry to name: fixed; D10 keys them `question-<id>` and `driver-<row>-<time>`. Merged with the two findings below.
- fix-way-forward, MEDIUM, the orch cannot name a history entry from its notice: fixed; the `halt:` stamp rides the notice, and an orch-classified halt with no notice carries no key and dedups on the slug.
- fix-proposal, MEDIUM, a fold into the same entry reads as the old halt: fixed; the key includes `repeats`.
- fix-consistency, LOW, the operator column's "always" disagrees with `false`: fixed; the column header names `true` and `unattended`.
- fix-consistency, LOW, the support log cites round 1 under "No rounds yet": fixed; the line no longer names a round.
- fix-consistency, outside its lens, the Bouncer's auto-accept and soft-cap accept can land open BLOCKING findings: fixed.
  D6 runs every perch as with `blocking_escalates` set under `unattended` (`internal/shedadapters/bouncer.go`), and D6's Bound and criterion name the Bouncer.
- fix-way-forward, MEDIUM, a hold lifted by a blanket resume has no way forward: fixed; the hold entry stands through a resume with no slug.
- fix-proposal, LOW, a blanket lift turns a hold into a severe post: fixed by the same change.
- fix-proposal, MEDIUM, the held notice would fire for an unread reply: fixed; the held notice keeps the combined probe (F17).
- fix-proposal, LOW, F21's option delays a severe post by two intervals: fixed. D7 states the latency as the price of one severe post per halt.
  The other option needs the dobby to read the orch's outbound Slack mail, which no record names.

## Disputed

None.

## Deferred

None.

## Rulings

None.
