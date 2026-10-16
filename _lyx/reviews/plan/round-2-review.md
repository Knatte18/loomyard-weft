---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/03-github-per-request-deadlines.md
    summary: Card 3 turns selfreportengine's callTimeout into a package-level var so a test can shorten it, which PATTERN-test-speed's Time is injectable rule forbids
    fix: In card 3, keep `callTimeout` a const (its comment change stays) and add the Create target `plan:internal/selfreportengine#listOpenIssues` -> `func listOpenIssues(perRequest time.Duration) ([]Issue, error)`, placed after the Edit list. The Intent then says `ListOpenIssues` returns `listOpenIssues(callTimeout)`, the page loop deriving `context.WithTimeout(context.Background(), perRequest)` per page, and the held-page row in `TestListOpenIssues_PaginatesAndExcludesPullRequests` calls `listOpenIssues` with its short budget instead of reassigning `callTimeout`. Drop the sentence "`callTimeout` becomes a package `var`, so a test can shorten it" and the "sets `callTimeout` to 50 ms and restores it" clause. The combined overlay with F4's target passed `lyx loom validate-plan --overlay` with no new finding.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-github-per-request-deadlines.md
    summary: Card 3's held-page row runs through installGitHubClient's plain go-github client, which has no authRT, so the pause gate never holds page 2 and the row never reaches the hold it claims to pin
    fix: "In card 3's Intent, state that the held-page row builds its client with `githubclient.NewWithBaseURL(server.URL, nil)`, with `t.Setenv(\"GH_TOKEN\", ...)` and `t.Setenv(\"GITHUB_TOKEN\", \"\")` as `githubclient`'s own pause test does, and installs it through the `NewGitHubClient` seam in place of `installGitHubClient`. The row also asserts that the gate's `Sleep` ran once, so it fails when page 2 goes out unheld. No target changes."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-shuttle-conditioned-send.md
    summary: typeVerified carries sendVerified's retryLanded check, so a conditioned paste send after an earlier unlanded revive types nothing yet reports the send landed, and revive reports sent_no_turn_start for a line it never typed
    fix: "In card 4's Intent, move the `retryLanded` check out of the cut: `sendVerified` becomes `awaitIdleSession`, then `retryLanded` on the paste path, then `typeVerified`, so `typeVerified` holds only the paste and typing paths and `Runner.SendConditioned` never consults `pending-send.json`. Add one row to `TestRunner_SendConditioned` over a `pasteEngine`: a `pending-send.json` holding the same text with a `START` past its offset, then `START` and `APIERR`; the line is pasted and the result is `turnStarted` as scripted. The paste path is the one production's engine takes, so the row also covers it. No target changes: `sendVerified` and `conditionedsend_test.go` are already on the card."
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/05-orch-conditional-notice.md
    summary: Card 5 makes delivery depend on the orch's session state but leaves two comments saying nothing the watcher decides depends on it, in logIdleStateDisagreement's doc comment and in orchengine doc.go's session-state paragraph
    fix: "Add `internal/orchengine#Watcher.logIdleStateDisagreement` to card 5's Edit list after `internal/orchengine#Session`, and in its Intent state that its doc comment's clause \"nothing the watcher decides depends on the state\" and `doc.go`'s session-state paragraph (\"The watcher only logs it\" and \"no probe result, cycle, delivery or State write depends on the state\") now name the conditioned notice's drop as the one decision that reads the state. The overlay passed with F1's change."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/06-orch-watch-binary-reexec.md
    summary: Card 6 prescribes a state read error branch in restartSafe that no row reaches, and no input can reach it through Run, since tick's own LoadState fails first and Run calls the check only after a tick that succeeded
    fix: "In card 6's Intent, either name a row that reaches the state re-read error after a successful tick, or drop the branch. A reaching row would use a `fakeSession` hook during the tick that replaces the state file with a directory after the tick last saves it. To drop the branch, have `restartSafe` take the phase from the state the tick just ran on instead of re-reading it, and remove \"A state ... read error is not safe\" for the state half."
  - id: F6
    severity: LOW
    class: design
    location: _lyx/plan/03-github-per-request-deadlines.md
    summary: Card 3's held-page row passes only on a 20 ms wall-clock margin, a 50 ms budget with page 2 answering after 30 ms, which -race and gate-slot load can eat
    fix: "In card 3's held-page row, widen the budgets: a per-request budget of 200 ms, page 1 answered after 170 ms and page 2 after 60 ms. A shared deadline then leaves page 2 30 ms and still fails, and a fresh one gives it 140 ms of margin. With F1 applied, the budget is the argument passed to `listOpenIssues`."
---

### [BLOCKING] Card 3 makes `callTimeout` a package-level var

**Location:** `_lyx/plan/03-github-per-request-deadlines.md`, Intent paragraph beginning "`callTimeout` becomes a package `var`".
**Issue:** `PATTERN-test-speed`'s "Time is injectable" section says a production timeout a test would shorten "is a struct field or a parameter, never a package-level `var`".
Card 3 changes `internal/selfreportengine/selfreport.go`'s `const callTimeout = 30 * time.Second` into a `var` solely so `TestListOpenIssues_PaginatesAndExcludesPullRequests` can set it to 50 ms and restore it.
The package already serializes its tests on the `NewGitHubClient` seam, but that does not exempt the new var.
The rule names no such exception, and every listed constraint violation is BLOCKING.

### [MEDIUM] Card 3's held-page row never goes through the pause gate

**Location:** `_lyx/plan/03-github-per-request-deadlines.md`, the paragraph on `TestListOpenIssues_PaginatesAndExcludesPullRequests`'s second case.
**Issue:** The pause hold lives in `internal/githubclient/transport.go`'s `authRT.RoundTrip`.
`internal/selfreportengine/selfreport_test.go`'s `installGitHubClient`, which the existing test uses, builds `github.NewClient(nil).WithAuthToken("test-token")`, a client with no `authRT`.
The card says the row "installs a pause gate" but never says the client changes.
An implementer reusing the helper gets a page 2 that `pauserecord.Network` never sees, and the gate's `Sleep` never runs.
The row still passes, since a fresh 50 ms budget beats a 30 ms answer, so it pins the per-page deadline but not the decision record's acceptance criterion that "a several-request GitHub caller whose second request is held by a pause completes after resume".

### [MEDIUM] A conditioned send can report a line it never typed as landed

**Location:** `_lyx/plan/04-shuttle-conditioned-send.md`, "`typeVerified` is the typing half of `sendVerified`, cut out with no change: the paste path, ...".
**Issue:** In `internal/shuttleengine/run.go`'s `sendVerified`, the paste path first calls `retryLanded(sc, text)`, which returns true, typing nothing, when `pending-send.json` holds the same text and the events file shows any turn start past the recorded offset.
Cut out "with no change", `typeVerified` keeps that check, and `Runner.SendConditioned` then treats the send as landed and waits for a turn start past its own offset.
Production's engine (`claudeengine`, which implements `PasteSubmits`) takes the paste path, and `pasteVerified` writes that record on every `ErrSubmissionNotLanded` or `ErrSentTextInDraft`.
The design itself creates the scenario.
A first `revive` does not land, and with `keepUnlanded` its `ReviveLine` stays in the box and a pending record is written.
The operator submits that text, the turn hits the outage again, and the session reads idle-stalled `api-error` with an empty box.
The next `revive` passes `conditionedReading`, `retryLanded` returns true, nothing is pasted, no turn start follows, and the report says `sent_no_turn_start`.
D5's report contract ("sent, sent with no turn start seen, not landed ...") then misstates what happened, and the stalled session gets no line.
`TestRunner_SendConditioned` runs over `pickerEngine`, which is no `TextPaster`, so no row reaches the paste path.

### [LOW] Card 5 leaves two comments claiming the session state decides nothing

**Location:** `_lyx/plan/05-orch-conditional-notice.md`, the Edit list and the `doc.go` sentence.
**Issue:** `internal/orchengine/watcher.go:603`, in `Watcher.logIdleStateDisagreement`'s doc comment, says "nothing the watcher decides depends on the state".
`internal/orchengine/doc.go:151` says "The watcher only logs it", and `doc.go:154` says "no probe result, cycle, delivery or State write depends on the state".
After card 5, `deliverNotice` reads the state and drops a conditioned notice on it, so all three become false.
The card updates the `Session` interface comment and `doc.go`'s "Notices" section only.
`lyx quarry resolve` places `Watcher.logIdleStateDisagreement`'s span from line 598, covering line 603.

### [LOW] Card 6's state re-read error branch is unreachable by any row

**Location:** `_lyx/plan/06-orch-watch-binary-reexec.md`, "A state or record read error is not safe, so the check waits for the next tick."
**Issue:** `Watcher.Run` calls the check only after a tick that returned no error.
`Watcher.tick` begins with `LoadState(w.paths)` and returns its error, so an unreadable state file fails the tick before `restartSafe` runs.
The state half of that branch is reachable only if the state turns unreadable between the tick and the re-read, and the card names no row or input for it.
The record half is reached: `pauseTick` tolerates an unreadable record, so the row with a directory at `RecordPath` gets through the tick to `restartSafe`.

### [LOW] Card 3's held-page row rests on a 20 ms margin

**Location:** `_lyx/plan/03-github-per-request-deadlines.md`, the held-page row's timings.
**Issue:** With a 50 ms per-request budget and page 2 answering after 30 ms, the passing case has 20 ms of slack against real context deadlines.
A -race build under a loaded gate slot can lose that to scheduling, which makes the row fail intermittently.
Context deadlines cannot be faked, so the margin is the only lever.
Card 2's rows run on about 50 ms of slack, which is tighter than ideal but not raised here.

## Rejected

- Focus item 1, card 1's `held` for an uncovered gated call: not a defect.
  `Network` wraps the `Waiter`'s sleep in a closure that notes it ran, and `held` stays zero unless that closure ran, so an uncovered call gets exactly zero under a real or a fake clock.
  The card says how `Network` tells that the wait slept: `WaitUncovered` returns only an error, hence the wrapper.
  In `internal/pauserecord/hold_test.go`'s `TestNetwork`, the first `Network` call of the SetGateSlug row runs with an empty slug against a slug-`a` entry, so it is uncovered.
  With a fake `Now` advancing 1 s per call, an implementation measuring elapsed time unconditionally reads 1 s there and fails the row.
- Focus item 2, card 5's notice naming: not a defect.
  A conditioned name ends in `.cnotice`, and `noticeNames` today keeps only names whose suffix is `.notice`.
  `.cnotice` does not end in `.notice`, so a watcher built before this change never lists, types or caps a conditioned notice; `DropNotices` removes the whole directory either way.
  Every name keeps its arrival-time prefix, so one lexical sort orders both kinds, and the `.tmp-` files match neither suffix.
  The bound on delivery is stated: the watcher stays the only writer, delivery keeps today's phase and idle-probe gates, and a state that no longer matches drops the notice.
- Focus item 3, card 4's pane-too-short hold: not a defect.
  `judgeReadiness` reports too-short and no-input-box as reasons of one held reading, and `conditionedReading` maps every held reason to one `*SessionBusyError` branch.
  The no-input-box, draft and stray-character rows reach that branch, and `internal/shuttleengine/session_test.go` already pins the too-short reason.
- Focus item 3, card 5's state read error: the `sessionStateErr` row reaches it.
  Card 9's state read failure and process error, and card 10's state read failure, unknown slug and agent caller rules, each have a named row whose input no upstream check rejects.
