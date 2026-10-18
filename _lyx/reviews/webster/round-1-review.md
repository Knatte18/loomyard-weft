---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/orchengine/watcher.go:479
    summary: readMailbox saves a stale state over the handler's work when the read acknowledgement fails
    fix: In readMailbox, when err is non-nil and the handler ran (w.repliesRead is true), reload the state with LoadState(w.paths) before calling creelFailed, so creelFailed saves the handler's state with Stuck set instead of the pre-handler copy; add a TestWatcher_CreelFailuresHoldTheCycle row where the handler moves the phase and the acknowledgement then fails, asserting the phase is kept and the reply is not acted on twice.
    origin: lens:code-correctness
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/orchengine/watcher.go:421
    summary: retryFreshLaunch fails the tick when ClearResumeNoticed fails after the withdraw removed the resume line, so a retry never sees the line again and the mark stays set
    fix: In retryFreshLaunch, treat a ClearResumeNoticed error like a withdraw failure, adding it to failures and skipping the release, so the field stays set; or run the clear before it can be lost by recording the withdrawn resume line in State until the clear succeeds. Add a TestWatcher_CreelFailuresHoldTheCycle row for a withdraw that removes a resume line while ClearResumeNoticed fails.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/orchcli/creelseam.go:49
    summary: creelSeam's Inject (text and command branch with ttl), ReadMailbox, Withdraw, Hold, Release and Post mappings have no test; TestCreelSeam_FoldsTheJournal reaches only InjectOutcome and Sent
    fix: Extend internal/orchcli/creelseam_test.go with rows over the fake creel client of internal/orchcli/fakecreel_test.go that call each remaining creelSeam method, asserting the request the client received (command versus text inject, ttl seconds, receiver name) and the converted CreelMessage and CreelWithdrawn values.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: design
    location: internal/creelcli/goclient.go:76
    summary: nothing pins that every GoClient method passes goCallTimeout, which the go-client-surface decision makes the contract
    fix: Add one table row per GoClient method in internal/creelcli/cli_test.go (beside the existing GoClient rows) against a listener that never answers, asserting each call returns a timeout error; inject a short timeout through an unexported field on GoClient so the test does not wait out 30 seconds.
    origin: lens:code-tests
  - id: F5
    severity: NIT
    class: consistency
    location: internal/battenshed/notice.go:296
    summary: comments in battenshed/notice.go, deps.go:47, teardown.go:69 and innerrun.go:334 still say a notice is queued, naming the retired orch notice queue; a notice is now posted as creel mail
    fix: Reword those comments to say the notice is posted as creel mail and that deliver reports it posted, keeping the behaviour unchanged.
    origin: lens:code-affected-set
---

### [MEDIUM] readMailbox saves a stale state over the handler's work when the read acknowledgement fails

**Location:** internal/orchengine/watcher.go:479
**Issue:** `readReplies` loads and saves its own state, and may move the phase and queue steps, before the client acknowledges the read.
When the acknowledgement then fails, `readMailbox` passes its pre-handler `st` to `creelFailed`, which saves it whenever `Stuck` differs, moving the phase back.
The messages stay unread, so the next tick reads the same reply again and starts the clearing steps a second time.

### [MEDIUM] retryFreshLaunch loses a withdrawn resume line when the mark clear fails

**Location:** internal/orchengine/watcher.go:421
**Issue:** The withdraw has already removed the resume line when `ClearResumeNoticed` fails.
A retry's withdraw answers an empty list, so the noticed mark is never cleared and the line is never posted again.
The failure also returns an error that fails the whole tick, where a withdraw failure only records a retryable failure.

### [MEDIUM] creelSeam's production mapping is mostly untested

**Location:** internal/orchcli/creelseam.go:49
**Issue:** The orch watcher's tests use `orchengine`'s own fake of the seam and `start_test` replaces the whole seam, so neither reaches the real adapter.
`TestCreelSeam_FoldsTheJournal` covers `InjectOutcome` and `Sent` only; the command-versus-text branch of `Inject`, the ttl conversion, the mailbox conversion and the withdraw conversion are behaviour the diff adds with no coverage.

### [LOW] No test pins goCallTimeout on every GoClient call

**Location:** internal/creelcli/goclient.go:76
**Issue:** Each of the `Call` sites passes the constant by hand.
`NonWaitingCallToAListenerThatNeverAnswersTimesOut` in `internal/creelengine/server_test.go` pins the client's bound, but no test shows each `GoClient` method uses it, and the go-client-surface decision relies on it to cap the lookups tender makes under the pause record lock.

### [NIT] Stale queue wording in battenshed comments

**Location:** internal/battenshed/notice.go:296
**Issue:** The comments describe the retired orch notice queue; the behaviour is a creel post.

## Rejected

- Both declared PATTERN changes landed with the declared text, including the verbatim background sentence; nothing to raise.
- The `PATTERN.md` shortenings of unrelated entries are authorised by card 8's overview-room text and keep each entry's name, test mark and background link.
- No commit carries a `Pattern-Departure:` trailer and no diff departs from a listed entry.
- Card 8 `assert-no-callers` holds: the remaining `notices.log` and notice-queue references are the deliberate retired-path handling in `lyx orch cleanup` and historical docs.
- A missing test for the watcher's retry of a failed start round: `TestWatcher_CreelFailuresHoldTheCycle` in `internal/orchengine/watcher_test.go` rows around line 2632 cover it.
- GoClient.Inject truncating a ttl under one second to the default (lens:code-correctness F3): no caller passes one, and the module is not a public API.
- saveFreshLaunch returning the second save's error after launching (lens:code-correctness F4): a state-file write failure that is reported loudly, with the session already bound.
- Second-round clearResumeNoticed keying on the reset state's GUID: the correctness lens found it harmless, since no entry names the new GUID.
- Dropped handoff rows in prompt_test.go (lens:code-affected-set): the lens could not show a gap, only a possibility.
- Two fakes of the creel seam in orchengine and orchcli (lens:code-tests): they implement different interfaces over unexported types, not a duplicated kit.
- Overlap of session_test and shuttlefake_test stamp rows: they pin different layers.
