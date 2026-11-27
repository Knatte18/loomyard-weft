---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/testkit/tmuxrec/recordings_test.go:206
    summary: One test file holds a tier0 test (TestRecordings_Replay) and a tier2 test (TestRecordings_RealTmux), and the file, read as tier0, references tmuxkit.Socket and exec.Command, against PATTERN-test-speed
    fix: Move TestRecordings_RealTmux, realTmux and every tmuxkit, exec and proc.KillPID use into a new tier2 test file of package tmuxrec_test, such as recordings_realtmux_test.go. Keep TestRecordings_Replay in recordings_test.go, and move the scenarios map and its helpers into a support file with no test function, such as scenarios_test.go, so both tiers' files share them. Leave every test file naming one tier.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/testkit/tmuxrec/session.go:156
    summary: The kit asserts beyond t.Fatalf on its own setup, because PlaySession's cleanup fails the test with t.Errorf for unanswered calls, which PATTERN-test-isolation's Enforcement section admits only for its named kits
    fix: Remove the t.Cleanup that calls t.Errorf from PlaySession, keeping its returned *Replay. In TestRecordings_Replay and in newPRSessionWorld (internal/landingshed/prreview_session_test.go), keep the replay PlaySession returns and add a t.Cleanup that fails the test when replay.Unanswered() is non-empty, naming the recording and the calls.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/testkit/tmuxrec/session.go:69
    summary: "The tier0 replay depends on the host: recordings hold the recording machine's exec.LookPath result for the shell (set-option default-shell /usr/bin/bash) and Linux-only argv, so on a host whose PATH resolves bash elsewhere, or on Windows, every replayed set-option or list-sessions call goes unanswered and every landingshed session test fails at tier0"
    fix: "In newSession, bind the shell path reed resolves (exec.LookPath of cfg.Shell, as windowsize.go:19 does) as a placeholder such as {shell} in Bindings, so placehold and bind carry it, and record the scenarios again. Make PlaySession skip the test with t.Skipf when runtime.GOOS differs from the recording's Source.OS, naming the recording's OS."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxrec/tmuxrec.go:326
    summary: The read fallback in Replay.answer scans back across recorded mutations, so a read recorded only before a change is answered after that change with the old server state, which contradicts its doc comment
    fix: In Replay.answer's backward loop, stop at the first call at or before r.next-1 whose read() is false, so a read is answered only from a recorded read after the latest recorded change. Have Call.read classify a chained argv (one holding a ';' element) as a read only when every subcommand of the chain starts with a read verb.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxrec/record.go:49
    summary: The recorder never stores a call's stdin and the replay ignores it, so a changed text in reed's paste call (load-buffer -b <buffer> -), present in every recording, still replays as a match
    fix: Add a Stdin string field (json tag stdin,omitempty) to Call, set it in Recorder.tmux from the stdin argument through the same placeholding as Out, compare it in Call.matches, bind it in bindCall, and pass stdin into the wanted Call in Replay.tmux. Record the scenarios again.
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/landingshed/prreview_session_test.go:248
    summary: "The registration check is looser than at the review base, len(s.registered) != 1 instead of the exact identity cmp.Diff([]string{\"fake-guid-1\"}, s.registered), and no commit carries a Loosened-Check trailer"
    fix: "Replace the length check at line 248 with an exact comparison against the replayed pane pid of the started strand: cmp.Diff([]int{1<<30 + 4}, s.registered), the session-start-stop recording's {pid:4} pane. Name the value as a constant with a comment pointing at that recording's pane."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxrec/recordings_test.go#TestRecordings_Replay
    summary: No test pins the replay's failure behaviour, the kit's central contract that a reordered, changed or unrecorded call is refused and reported, and a read is answered only from state as new as the last change
    fix: "Add a tier0 table test of NewReplay over a hand-picked cut of an embedded recording (picked and ordered, never edited), with rows: a mutating call out of its recorded order is refused and listed by Unanswered; a call with different args is refused; a read after a mutation with no recorded read after it is refused (pinning F4's fix); a read repeated before the next change is answered again."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxrec/tmuxrec.go:170
    summary: "Placeholding replaces raw substrings and digit tokens, so a short user or host name inside other text (such as tr in tr:pr-reviewer) or a pid equal to the uid in tmux-1000 and user-1000.slice is turned into a placeholder and corrupts a recording"
    fix: "In newSession, bind the uid (os.Getuid) as {uid} so its tokens are placeheld before pids. In Recorder.Recording, fail with an error when a {user} or {host} binding is shorter than four characters or occurs inside a fixed name of the session, such as Shortname or WorktreeName, naming the clash."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/testkit/tmuxrec/record.go:181
    summary: formatPIDs, pidPlaceholder, Bindings.placehold and placeholdCall run only under the tier2 test, whose readings compare holds no pid or path, so a placeholding defect shows only when a rewritten recording is next replayed
    fix: Add a tier0 table test in package tmuxrec (an internal _test.go file) of formatPIDs and placeholdCall over literal format and output lines copied from the recordings, with rows for a pane_pid field, the server pid, a path under a nested bound path and a number that is no kept pid.
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxrec/recordings_test.go:135
    summary: The session-ends scenario's poll loop ends silently when no read finds the pane dead within paneEndPolls, so a rewrite with -tmuxrec.write saves a recording with no ended step and nothing fails
    fix: After the poll loop in the session-ends scenario, call t.Fatalf when no reading found the strand dead, naming paneEndPolls and the last reading.
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxrec/recordings_test.go#TestRecordings_Replay
    summary: Nothing checks that a replay used every recorded change, so a scenario that stops making a trailing recorded mutation, such as its last kill-pane, still passes the tier0 replay
    fix: "Export a Replay method Remaining() []string that names each recorded non-read call at or after the replay's place, and in TestRecordings_Replay fail when it is non-empty after the scenario ran. Leave the landingshed rows, which replay only part of a recording, without that check."
  - id: F12
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: pattern/PATTERN-test-stand-ins.md:67
    summary: The diff adds a rule-document paragraph naming tmuxrec as the stand-in below reedengine, which no decision and no declared PATTERN change licenses and no traced change forces
    fix: Remove the two-line paragraph that opens with internal/testkit/tmuxrec from pattern/PATTERN-test-stand-ins.md.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:2105
    summary: The testkit section of docs/overview.md describes each kit but leaves out the new tmuxrec kit
    fix: After the shedcheckkit line in docs/overview.md's testkit list, add a line stating that internal/testkit/tmuxrec replays recorded tmux, process-table and user-manager answers below reedengine, through reedengine.NewOver.
  - id: F14
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:40
    summary: The change description names the tooling, with 'The entry asked for' naming the board entry and 'another branch in flight' naming another task, and its Merge danger blast radius names only reedengine though the diff also changes loomcli's production wiring
    fix: "In _lym/landing/summary.md, reword line 40 to state the request without naming the entry, such as \"A recorded reed and creel session stand-in was asked for.\", reword line 50 to \"envkit changes elsewhere first\", and make the blast radius at line 57 read `reedengine`, `loomcli` (its landing wiring)."
  - id: F15
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/reedengine/lock.go:120
    summary: The empty-scrub panic names New even when NewOver is the caller
    fix: "Change the panic text at internal/reedengine/lock.go:120 to \"reedengine: NewOver needs a non-empty EnvScrub\", or to name both New and NewOver."
---

### [BLOCKING] A test file names two tiers, and its tier0 reading admits spawns

**Location:** `internal/testkit/tmuxrec/recordings_test.go:206`
**Issue:** `TestRecordings_Replay` opens with `tiergate.Require(t, 0)` and `TestRecordings_RealTmux` with `tiergate.Require(t, 2)` in the same file.
`PATTERN-test-speed` says "The functions of one test file name one tier".
`impactset.FileTier` takes the lowest tier, so the file counts as tier0, and a tier0 file may not reference `exec.Command` or any `tmuxkit` export other than `Main`.
This file references both, through `realTmux` and `tmuxkit.Socket`.
Both guards carry `//lym:only parent-align finalize` (`cmd/lym/tiertags_test.go:124-125`, `cmd/lym/tierpurity_test.go:87-88`), so no card gate ran them, and the break would surface at landing.

### [BLOCKING] The kit asserts outside its setup

**Location:** `internal/testkit/tmuxrec/session.go:156`
**Issue:** `PlaySession` registers a cleanup that calls `t.Errorf` when the replay left calls unanswered.
`PATTERN-test-isolation`'s Enforcement section says "A kit asserts nothing beyond `t.Fatalf` on its own setup", and names its exceptions: `shedfake`, `envelope` and `scankit`.
`tmuxrec` is not among them, and the diff does not move the check into the tests.

### [MEDIUM] The tier0 replay depends on the host's PATH and OS

**Location:** `internal/testkit/tmuxrec/session.go:69`
**Issue:** The recordings were made on Linux and hold `set-option -t =pr-session: default-shell /usr/bin/bash` and the same for `default-command` (`session-start-stop.json:304-321`).
Reed takes that path from `exec.LookPath(shell)` (`internal/reedengine/windowsize.go:19`), which `Outside` leaves real.
Suppose a host's PATH puts `/bin` before `/usr/bin`, or the host has no `/usr/bin/bash`.
Then the args reed sends differ from the recorded ones, and the call goes unanswered.
On Windows, `serverSpawnArgv` and `listSessionsArgv` (`tmuxconf.go:168-172`) build different argv, so no call matches.
Each case fails every `landingshed` PR-Review session test and the kit's tier0 test, on a tier the repository runs on every card gate.

### [MEDIUM] A read after a change can be answered with the state before it

**Location:** `internal/testkit/tmuxrec/tmuxrec.go:326`
**Issue:** The backward loop in `Replay.answer` returns the latest recorded matching read before the replay's place, with no bound at the latest recorded change.
Suppose reed makes a read after a change, and that read was recorded only before the change.
The replay then answers it with the server's earlier state instead of failing.
This contradicts the comment "since nothing changed since that answer".
`Call.read` judges a chained argv by its first verb alone, which risks the same misclassification.

### [MEDIUM] The paste text is neither recorded nor matched

**Location:** `internal/testkit/tmuxrec/record.go:49`
**Issue:** `Recorder.tmux` drops `stdin`, and `Replay.tmux` (`tmuxrec.go:260`) ignores it.
`Call.matches` therefore compares args only, so reed's `load-buffer -b <buffer> -` paste (`internal/reedengine/io.go:147-151`) matches whatever text reed types.
Every recording holds that call.

### [MEDIUM] The registration check was loosened without a trailer

**Location:** `internal/landingshed/prreview_session_test.go:248`
**Issue:** At the review base, the first row asserted exactly which strand was registered, through `cmp.Diff([]string{"fake-guid-1"}, s.registered)`.
It now checks only `len(s.registered) != 1`, so registering any pid passes, such as the server's or another pane's.
No commit of the range carries a `Loosened-Check:` trailer.

### [MEDIUM] No test pins the replay's refusals

**Location:** `internal/testkit/tmuxrec/recordings_test.go#TestRecordings_Replay`
**Issue:** The package's only tier0 test replays scenarios that match their recordings.
No row drives a reordered, changed or unrecorded call, a stale read, or the `Unanswered` report.
So a replay that answered any call from any match would keep every test green.

### [LOW] Substring and digit-token placeholding can corrupt a recording

**Location:** `internal/testkit/tmuxrec/tmuxrec.go:170`
**Issue:** `Bindings.placehold` replaces raw substrings.
A user or host name such as `tr` therefore rewrites `tr:pr-reviewer`.
`pidPlaceholder` (`record.go:181`) replaces every digit token that equals a kept pid.
A pid equal to the uid in `tmux-1000` (`session-start-stop.json:332`) or in `user-1000.slice` (`:838`) is therefore placeheld.
The uid itself, a machine-specific value, stays in the recordings unplaceheld.

### [LOW] Placeholding has no tier0 coverage

**Location:** `internal/testkit/tmuxrec/record.go:181`
**Issue:** The placeholding functions run only under `TestRecordings_RealTmux`, which compares `Readings` alone.
Readings hold no pid or path, so a placeholding defect passes that compare.

### [LOW] The pane-end poll ends silently

**Location:** `internal/testkit/tmuxrec/recordings_test.go:135`
**Issue:** When no read finds the pane dead within `paneEndPolls`, the loop ends with no `ended` reading and nothing fails.
A rewrite with `-tmuxrec.write` then saves a recording that lacks that step.

### [LOW] Unused trailing recorded changes go unnoticed

**Location:** `internal/testkit/tmuxrec/recordings_test.go#TestRecordings_Replay`
**Issue:** The replay reports only calls it could not answer.
A recorded change the code no longer makes at the end, such as the last `kill-pane`, leaves the tier0 replay green.
Only the readings comparison would catch it.

### [LOW] An unlicensed PATTERN paragraph

**Location:** `pattern/PATTERN-test-stand-ins.md:67`
**Issue:** The diff adds a paragraph naming `tmuxrec` as the stand-in below `reedengine`.
No decision asks for it, the directive declares no PATTERN change, no traced change leaves the background stale, and no `Pattern-Departure:` trailer explains it.

### [LOW] The overview's kit list lacks tmuxrec

**Location:** `docs/overview.md:2105`
**Issue:** The testkit section names each kit with a line on what it holds (`docs/overview.md:2092-2105`), and the new kit is missing from it.

### [LOW] The change description names the tooling and understates the blast radius

**Location:** `_lym/landing/summary.md:40`
**Issue:** "The entry asked for…" names the board entry, and "envkit is being changed by another branch in flight" (`:50`) names another task; both are tooling names the rubric bars from the description.
The blast radius at `:57` names `reedengine` alone, though the diff rewires `loomcli`'s landing seams in `internal/loomcli/sharedbootstrap.go:439-447`.

### [NIT] The panic names the wrong constructor

**Location:** `internal/reedengine/lock.go:120`
**Issue:** `NewOver` panics with "reedengine: New needs a non-empty EnvScrub", which misnames the caller when `NewOver` is called directly.

## Rejected

- The extra `Outside` fields `CapMemory` and `NewGUID` as unrequested scope: the tier0 replay needs them, to keep pane creation off the user bus and to mint the guids the recordings name.
- The `"testdata"` literal repeated in `session.go` and `recordings_test.go`, under the convention that a name which can go stale is one named constant: Go's `testdata` directory name does not go stale, and the embed directive must spell it anyway.
- The told-tmux spawn dropping the env scrub, `cmd.Dir` and the debug flags: a hook addresses its own server, and the recorder's server comes from `tmuxkit.Socket`, pre-started with the scrubbed environment, so none of these reaches a recording.
- The summary's claim that a replay never reads or signals a process of the machine: replayed pids lie above the kernel's pid range, so the removal path's process-table reads find no process, and the memory cap and pane age go through the replay.
