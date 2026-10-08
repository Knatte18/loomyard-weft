---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/battencli/wire.go:623
    summary: ReadDecision, StopReport and AttachDir each resolve the task worktree through lyxcwd.ResolveWorktree (a git rev-parse process), and the wait calls them on every poll_interval_s check, breaking the rate-of-reads decision
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/reedengine/lifecycle.go:921
    summary: Down now reads reed state through the strand-window seam to find its reap roots, so a corrupt reed.json silently skips the pane process-tree reap, and Down's doc still says it loads no state
  - id: F3
    severity: LOW
    class: consistency
    location: internal/battenshed/innerrun.go:623
    summary: The running arm still writes the wait reason sleeping before the next bounce, though the row no longer bounces between polls
  - id: F4
    severity: LOW
    class: design
    location: contracts/stencils/shed/shed-template-driver-notify-watched.md:3
    summary: The watched stencil says something else watches the run and tells the parent about every stop, but card 3 requires it to say nothing of who announces a stop
  - id: F5
    severity: NIT
    class: consistency
    location: internal/reedengine/panebin.go:3
    summary: Changed comments keep mid-sentence column breaks and put two sentences on one line, and PATTERN-pane-binary-resolution still names launchStrandLocked as the seam's only call site
---

Range reviewed: `git diff $(git merge-base main HEAD)..HEAD`.
`_lyx/shed/lifecycle-waits/status.json` gives `product.parent` = `main`, and there is no `_lyx/loom/rework/` directory, so the rubric's no-rework-round branch applies.
Evidence gathered: `go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, `go test ./...`, `go test -tags integration` on battencli, battenshed, orchengine and shuttleengine, `go test -tags "integration tmux"` on reedengine, and `lyx loom lint-comments --base <merge-base>` all pass.
Passing tests do not cover the findings below: the notice-step tests fake `StopReport`, `AttachDir` and `ReadDecision`, so the cost behind those seams in `battencli/wire.go` is never exercised.

### [BLOCKING] Process-costing reads run on every poll check

**Location:** `internal/battencli/wire.go:623` (`ReadDecision`), `internal/battencli/wire.go:399` and `:585` (`readStopReport`, wired as `StopReport`), `internal/battencli/wire.go:603` (`AttachDir`); callers `internal/battenshed/innerrun.go:739` (`awaitingStep`), `internal/battenshed/notice.go:243` and `:342` (`noticeStep`, `stateChangeReady`).
**Issue:** the plan's rate-of-reads decision lets file stats and small file reads run on every `poll_interval_s` check, and allows anything costing a process at most once per `notice_probe_s`.
Card 5 names the decision record among the cheap per-check reads, and card 7 treats the park marker the same way.
Each of the three seams starts with `taskWorktreeLocation`, which calls `lyxcwd.ResolveWorktree`, which calls `gitexec.Run(rev-parse --show-toplevel)`: a git process per call.
With `poll_interval_s` at 2 s, this means:
- an awaiting child runs `ReadDecision` on every check, so a git process every 2 s for as long as the child awaits a decision (hours or days);
- a halted child's pending notice runs `readStopReport` and then `AttachDir` on every check until the notice is queued, and with no orch strand recorded the notice never is, so it spawns git every 2 s indefinitely (`deliver` spaces only the `Notify` and `OrchStrandRecorded` calls, after `AttachDir` has already run);
- `undecided()` repeats `ReadDecision` on every check of an awaiting relay wait.

Before this change `ReadDecision` ran once per 30 s Call, so the in-call wait multiplies the cost 15 times.
`internal/battenshed/doc.go` also lists "the notice step" among the work that runs at most once per `notice_probe_s`, which is not true of the per-check `noticeStep` path.
**Fix:** make the file reads actually cheap.
Resolve the task worktree location once per wiring (memoize `taskWorktreeLocation`'s successful result inside the `wire` closures, since a present worktree's location does not change), or have the seams take a path computed once, so `ReadDecision`, `StopReport` and `AttachDir` are plain file reads.
Alternatively gate the process-costing seams on the probe-due clock.
Add a test that counts the location resolutions over a long awaiting wait and a long pending-notice wait, and correct the `doc.go` sentence.

### [MEDIUM] Down reads reed state to find its reap roots

**Location:** `internal/reedengine/lifecycle.go:921` (`sessionReapRootsLocked`), `internal/reedengine/windowtarget.go:68` (`storedStrandWindowTarget`), `internal/reedengine/lifecycle.go:750` (`Down` doc).
**Issue:** card 9 keeps the current-window fallback for the session-kill reap, because that reap ends the whole session.
The engine-less `reapSessionVia` follows that, but `Engine.Down` reaches `paneProcessTreePIDsLocked` and `sessionReapRootsLocked`, which now call `listStoredStrandPanes` and so `LoadState`.
`Down` is documented as the escape that loads no state, and it is the verb that still works with a corrupt `reed.json`.
With a corrupt file `LoadState` errors, `sessionReapRootsLocked` returns nil, and `Down` kills the session without reaping the pane process trees; the doc comment explains why that reap matters (on Windows a deeper descendant holds the worktree directory).
The resolved strand window also covers only that window's panes, so a batten window's process tree is not reaped when the session is killed.
No test reaches `Down` over a corrupt state.
**Fix:** take the reap roots from the session-wide pane listing (`list-panes -s`, every window of the session) so they need no state and cover the batten window, or fall back to it when the state cannot be read.
Add a case to the `Down` coverage for a corrupt `reed.json`, and update the `Down` doc comment.

### [LOW] Wait reason still says the row sleeps before the next bounce

**Location:** `internal/battenshed/innerrun.go:623` and `:625`; pinned by `internal/battenshed/innerrun_test.go:426`.
**Issue:** the running arm reports `inner shed run still running; sleeping %s before the next bounce` through `reportStuck`, which `lyx batten status` shows.
After card 5 the row waits inside its call and returns only on an event, so there is no next bounce and no sleep between bounces; the text misdescribes what the child is waited on for.
**Fix:** reword the reason to say the row is watching the child (and keep the reviewer-wait note), and update the pinning test.

### [LOW] Watched stencil names who announces the stops

**Location:** `contracts/stencils/shed/shed-template-driver-notify-watched.md:3` and `:12`.
**Issue:** card 3 requires both notify stencils to stay recipe-blind and says "the watched one says nothing of who announces a stop".
The stencil opens with "Something else watches this run and tells the parent about every stop" and closes with "those stops reach the parent by other means".
Both sentences describe who announces the stops, and "every stop" is also loose: the driver itself relays an `awaiting` stop's `parent_notice`, and the batten notice for a parked stop can wait up to three minutes.
**Fix:** state only the driver's own rule (message the parent in these three cases, otherwise nothing, and never announce that the run stopped, halted or finished) without describing the other announcer, and keep the claims in `claims_test.go` matching.

### [NIT] Changed comments keep column wrapping; pattern text is stale

**Location:** `internal/reedengine/panebin.go:3` (file header), `internal/reedengine/doc.go:706` (bullet "all targeted at the strand window"), `internal/reedengine/doc.go:685`, `pattern/PATTERN-pane-binary-resolution.md` (first bullet).
**Issue:** `PATTERN-comment-line-breaks` asks for one sentence per line on changed comments.
The `panebin.go` header sentence the edit touched still breaks mid-sentence ("joins it" / "onto a strand's launch command") and its third line now carries the end of one sentence and the start of the next.
The two `doc.go` bullets the edit touched keep the legacy mid-sentence breaks ("per the Strand" / "window and Session targeting grammar").
`lyx loom lint-comments` passes because it flags only new breaks that end no sentence, so these stay review findings.
The header and `PATTERN-pane-binary-resolution.md` still say `launchStrandLocked` is the composition's only call site, though `window.go` now calls `composeWindowCommand` too.
**Fix:** rewrap the touched sentences one per line, and reword the header and the pattern bullet to name both call sites.

## Checked and not findings

- Rate-of-reads for the in-call reads that are cheap: the status stat, the pause flag, the done-seen marker and the halt markers run per check; `DriverAlive`, `DriverStrand` revive, `ReviewWait`, the forced decode, `Activity`, the watched-marker refresh, the not-parked resume retry and the notice `Notify` and `OrchStrandRecorded` calls are gated by `lastProbe` or `lastTry` against `noticeProbe`.
  The clock lives in `childWait`, per Call rather than per producer; every Call ends on an event, so this does not break the decision.
- `battenshed` production imports are `shedengine` and `logger` only, and the new readings arrive through `InnerRunDeps` and `SeedChildDeps` seams with `battenshed`'s own `AgentActivity` type.
- Episode state: `since` and the sent keys live in `noticeEpisode` across Calls, only a first sight reads `since` from the status file, and `TestNotice_SameStateRewriteMovesNeitherSinceNorTheKey` and `TestNotice_RestartDoesNotRenotify` pin it.
- Cards 1 and 2: `tickCompacting` completes on `CompactedSince`, and `removeSatisfiedRequest` removes a same-mode request not after the boundary, or after a confirmed `/clear`, and leaves the other mode and a later compact request.
- Card 9 departs from the plan in one place, openly: the pane-generation read keeps `=<session>:` where the card says `=<session>`.
  I confirmed on tmux 3.6 that the bare `=<session>` form expands `#{session_name}` and `#{session_id}` to empty, so the departure is correct and `doc.go` and `generation.go` say why.
- Test economy: each new test either covers a behavior no existing test covers or extends the named existing test; `TestNotice_NotifyErrorDoesNotChangeOutcome` was folded into the delivery table, whose harness `run` asserts the Stuck outcome.
  The api-error notice, the watched marker (`stepMarkWatched_HoldsTheMarkerOnlyWhileItsNoticesReachTheDriversParent`) and the wait events (`TestInnerRun_WaitEndsOnlyOnAnEvent`) are covered.
- Per-card mechanical checks: the cards carry only `Create` and `Edit` groups, so no `Rename` or `Delete` check applies; the `Edit` blast radius is covered by the full build and vet under every tag, which compile every caller of the changed `Notify`, `listPanes`, `driverPrompt` and `pinGeometryOptionsLocked` signatures.
