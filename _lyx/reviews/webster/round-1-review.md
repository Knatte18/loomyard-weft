---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/attach.go:12
    summary: New and changed Go comments are hard-wrapped at a column and carry several sentences per line, against PATTERN-comment-line-breaks
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/orchengine/state.go:140
    summary: A watcher tick's whole-state save overwrites the WatcherStopping record the signal goroutine just wrote, so start can still read a dying watcher as live
  - id: F3
    severity: LOW
    class: consistency
    location: contracts/stencils/orch/orch-template-reload.md:1
    summary: The reload-pointer stencil comment still says the orch skills are loaded again before the pointer, which no longer holds after a compaction
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, since `_lyx/shed/bugfix-sessions/status.json` names `product.parent: main` and no `_lyx/loom/rework/` round exists.
I read the diff card by card against `_lyx/plan/`, ran `go build ./...`, `go vet ./internal/...` with and without the `integration tmux llm` tags, and `go test` over the touched packages; all pass.

Focus questions settled with no finding:

- Typed text-plus-submit: every Claude sequence (`ComposeSend`, `ModelSwitchSequence`, clear, compact, reload-plugins) is now text with a settle, then a bare `Enter`; no production code sets `PaneInput.Submit`, and `reed.SendText` is reached only from `playInputs`.
  `sendVerified` stops after `sendExtraEnters` (2) extra Enters and then fails naming the pending input.
  The `submit_settle_ms` config key is the operator-approved exception recorded in the decision record, so it is not a scope finding.
- Orch pane typing: `/reload-plugins`, the skills step and the pointer are all typed by the watcher through `Session`, idle-gated; only `tickClearing` passes `skipsSkills` false.
- Strand removal (cards 5-7): `Attach` and `AttachGated` remove only respawn-eligible, tracked, live strands of the same output set; `AttachIfLive` removes nothing and the two Bouncer wait-only probes use it; `awaitTerminal` removes the strand on every terminal digest before the refusals; `RemoveRecoveryStrands` runs only for `--to start`.
- Cards 8-9: a step error records the voucher from the status file except on `ErrShedBusy`; the note names the lock and voucher readings and lists the whole history.
- Cards 14-16: no `switch-client`, `attach-session` or handover code remains under `loomcli` or `orchcli`; `--no-attach` is a hidden no-op on both start verbs; `WaitWatcherGone` is bounded at three poll intervals and does not wait without a stopping record (but see F2).
- Cards 17-18: every round's gate spec ends with `ReviewGateEntry`, the re-attach hop appends it on a clone, and the verb calls `CheckReviewFile`, the same function the gate calls.
- Per-card mechanical checks: the `Rename` and `Delete` targets are gone (no `decideHandover`, `OwnsTmuxEnv`, `ClientSession`, `SwitchClientArgv`, `handover.go`, `RecentHistory`, `recentHistoryRows` or `StartAttachOnly` remains), and the build is clean.
- Test economy: each new test or row sits where the card says; I found no duplicate of an existing test's coverage and no uncovered added behavior besides the one named in F2.

### [BLOCKING] Hard-wrapped, multi-sentence-per-line comments in new and changed Go comments

**Location:** `internal/shuttleengine/attach.go:12-15`, `:39-40`, `:49-56`, `:60-61`, `:67-68`, `:172-173`; `internal/shedadapters/burler.go:503-509`; `internal/burlerengine/doc.go:279-287`; `internal/loomcli/bootstrap.go:60-61`; `internal/loomcli/start.go:366-369`, `:522-525`.
**Issue:** `PATTERN-comment-line-breaks` requires, for any Go comment written or changed, semantic line breaks: one sentence per line, no column limit.
The comment blocks above were added or reflowed by this diff at about a hundred columns, break mid-sentence, and in places hold two sentences on one line (for example `// ... set: re-running a producer supersedes the session it halted. AttachIfLive is the same probe with that`).
The surrounding older text is wrapped the same way, but the repository's own constraint outranks a convention inferred from neighbours, and the rubric says a line width is never inferred from surrounding code.
Other new comments in the same diff (`doc.go` of `shuttleengine`, `orchengine`, `websterengine`, `claudeengine/session.go`, `loomcli/halt.go`) follow the rule, which shows it is practicable.
**Fix:** Rewrite each listed block with one sentence per line and a break at an internal independent-clause boundary where a sentence is long, leaving the wording as it is.

### [MEDIUM] A tick's whole-state save can erase the `WatcherStopping` record

**Location:** `internal/orchengine/state.go:140` (`saveStateForStrand`), reached from `Watcher.save` in `internal/orchengine/watcher.go:135`; the record is written by `recordStoppingOnCancel` in `internal/orchengine/watchloop.go`.
**Issue:** Card 16 has the stopping record land "before the in-flight tick finishes", so that `lyx orch start` sees a signalled watcher as stopping.
`Watcher.tick` loads the state at its start and later writes the whole struct back through `saveStateForStrand`, which returns the caller's `s` unchanged from inside `UpdateJSON`.
When the signal arrives mid-tick, the goroutine sets `WatcherStopping` true; the tick's own save, made from the state it loaded before the signal, then writes `WatcherStopping` false again.
Saves occur on any tick that reads a turn end, enters a phase or moves a step, which are exactly the ticks that run long enough to be caught in flight.
The record is then gone while the watcher still holds `watch.lock`, so `WaitWatcherGone` probes once and `start` reads the dying watcher as live: the restart race the card exists to close.
`TestRun_RecordsStoppingBeforeTheInFlightTickFinishes` does not catch it, because its blocked tick never reaches a save.
**Fix:** In `saveStateForStrand`, carry the persisted `WatcherStopping` over when a record is found (`s.WatcherStopping = cur.WatcherStopping`), so only `recordExit("")` and the goroutine change it.
Extend `TestRun_RecordsStoppingBeforeTheInFlightTickFinishes` so the blocked tick goes on to a save (a turn end read on that tick) and the final `WatcherStopping` assertion proves the record survives it.

### [LOW] Orch reload-pointer stencil comment names a skills reload that no longer happens after a compaction

**Location:** `contracts/stencils/orch/orch-template-reload.md:1`, and `contracts/stencils/orch/orch-template-role.md:3`.
**Issue:** The reload pointer is typed after an auto-compaction, and the comment says it comes "once the orch skills are loaded again".
Card 13 drops the skills step for compactions: the sequence is now `/reload-plugins` then the pointer, and only `/clear` reloads skills.
The role stencil's header says the orch's skills are loaded by lyx before the pointer arrives, which is likewise only true after `/clear`.
**Fix:** Say in the reload stencil's comment that the pointer follows `/reload-plugins` after an auto-compaction, and in the role stencil's header that lyx loads the skills after `/clear` while a compaction keeps them.
