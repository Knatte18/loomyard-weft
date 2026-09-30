---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: internal/orchcli/refusal.go:7
    summary: A bare weft token in a production comment fails TestEnforcement_FabricVocabulary, so go test ./internal/lyxcwd/... is red on this branch
  - id: F2
    severity: MEDIUM
    location: internal/shuttleengine/session.go:1
    summary: New comments in shuttleengine and claudeengine break mid-sentence at a fixed column, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F3
    severity: MEDIUM
    location: internal/orchengine/doc.go:1
    summary: New comments across internal/orchengine break mid-sentence at a fixed column, against the Comment Line-Break Convention
  - id: F4
    severity: MEDIUM
    location: internal/orchcli/cli.go:1
    summary: New comments across internal/orchcli, tests included, break mid-sentence at a fixed column, against the Comment Line-Break Convention
  - id: F5
    severity: MEDIUM
    location: internal/orchcli/smoke_cycle_test.go:184
    summary: The smoke test's wait for the background-task turn to end is already satisfied before that turn starts, so card 14 step 3's wait is not implemented
  - id: F6
    severity: MEDIUM
    location: internal/orchcli/smoke_cycle_test.go:231
    summary: Neither open-risk observation can report a real answer - nothing asks the session for its SendMessage address, and BGDONE is checked before the background sleep can finish
  - id: F7
    severity: LOW
    location: internal/orchengine/doc.go:82
    summary: The package doc says any phase exceeding the handoff timeout aborts to idle, but a clearing timeout moves on to resuming
  - id: F8
    severity: LOW
    location: internal/orchengine/watcher_test.go:461
    summary: Card 8's every-return-to-idle case is asserted only for a completed resume and a resume timeout, not for the handoff abort, handoff timeout and resume-render abort returns
---

Review range: `_lyx/shed/hub-orch-strand/status.json` gives `product.parent` `main`; the range is `git diff $(git merge-base main HEAD)..HEAD`, merge base `85902381e`, 14 commits (cards 1–14).

Substrate runs: `go build ./...`, the plan's untagged and `integration` test sets and `go vet -tags smoke` all pass.
A full untagged `go test ./...` fails in `internal/lyxcwd` (F1).
The per-card Edit-group blast-radius checks were run: `go test -tags integration ./internal/fabriccli/... ./internal/hubforge/... ./internal/configsync/... ./internal/stencilstore/... ./internal/stencilcli/...` all pass with `orch` in `configreg.Modules()`, and the `cmd/lyx` pinned lists (helptree, seamsignature, sandbox_coverage, configstrictness, main.go `Long`) all carry `orch` consistently and pass.

Focus areas judged and found sound (not defects):

- `watcher.go` events cursor: the first tick starts at `PhaseEventsOffset` in a non-idle phase and at `LastInjectionOffset` in idle.
  Every return to idle (completed resume, resume timeout, handoff ask/timeout abort, resume-render abort) goes through `toIdle`, which persists the cursor and drops the in-memory newest event.
  Usage readings are taken only for turn ends read while the tick's phase is idle, so a pre-clear turn end is never read for usage and cannot start a second cycle.
- `PhaseInjected` and restarts: state is saved before every `Send`/`ClearSession`, a same-strand restart forces `PhaseInjected` false, the landed check re-sends at most once per tick and only on a passing probe with no turn end past entry, and `CycleCount` moves only on leaving `clearing` with a confirmed clear.
- Idle gating and error paths match card 8, including a handoff render error leaving state and the cycle request untouched and a `StrandAlive` error returning without a write.
- `watchloop.go`: bounded lock retries, lock held for the run, reset on a successful tick, `WatcherExit` recorded on cap, signal and strand-gone exits, and `WatcherLive` releases what it acquired.
- `shuttleengine/session.go` matches card 1's `toldErr`-first, missing-capability, offset and Debug-logging rules, and only `ClearSession` skips `requireReadyAgentPane`.
- claudeengine `IdleSession`, `ClearSessionSequence` and `transcriptContextTokens` match cards 2 and 3.
- `start.go`, `spawn.go`, `handover.go`, `session.go`, `cli.go` and `paths.go` match cards 10–12 and the orch-strand-identity, orch-agent-tool and orch-dir-layout decisions.

### [BLOCKING] Fabric-vocabulary leak in refusal.go

**Location:** internal/orchcli/refusal.go:7
**Issue:** The file comment says the name comparison "admits the prime's weft sibling".
`internal/orchcli` is outside the Fabric Vocabulary Invariant's owner set, and `TestEnforcement_FabricVocabulary/tree-scan` in `internal/lyxcwd` fails with `internal/orchcli/refusal.go: bare weft/warp token outside the owner set`.
The plan's verify command never runs `internal/lyxcwd`, so this regression passed every gate so far.
**Fix:** Say "the prime's fabric sibling", the phrasing `internal/battencli/refusal.go` uses for the same point.

### [MEDIUM] Mid-sentence comment breaks in shuttleengine and claudeengine

**Location:** internal/shuttleengine/session.go:1 (also engine.go:146–151, session.go:14–27 and 85–88, session_test.go:1–2 and 31; claudeengine/session.go:1–4, 18–39, 79–80, 107–109; claudeengine/usage.go:1–7, 19–21, 36–39)
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, with further breaks only at an independent-clause boundary.
These new comments wrap at a fixed column mid-phrase, e.g. "reading a live run's events from a caller-held / offset" and "the provider operations a caller needs to / cycle a live session's context".
**Fix:** Rejoin each sentence onto one line, breaking only at independent-clause boundaries.

### [MEDIUM] Mid-sentence comment breaks in orchengine

**Location:** internal/orchengine/doc.go:1 (also config.go:3–6, 55–56, 64–65, 73–74, 82–83; template.go:3–4; prompt.go:1–7; launch.go:1–2, 29–30, 42–43; state.go:3–4; watcher.go:1–8, 61–62, 161–162, 299–300; watchloop.go:1–2, 22–23, 77–78; seam_enforcement_test.go:1–4, 20–21, 40–42; config_test.go:1–2)
**Issue:** Same convention.
Examples: doc.go "hosted as an interactive / shuttle run", its list items ("then hands the terminal / over to reed's attach"), watcher.go "one poll of the idle check and the persisted / four-phase cycle", and config.go "which uses / internal/configengine.LoadOrTemplate".
The comma breaks before a participial or dependent phrase (config.go's accessor docs, launch.go:29, watcher.go:61, state.go:3) are not independent-clause boundaries either.
**Fix:** Rejoin each sentence, keeping breaks only at independent-clause boundaries.

### [MEDIUM] Mid-sentence comment breaks in orchcli

**Location:** internal/orchcli/cli.go:1 (also cli.go:3–5, 164–165; paths.go:1–3; refusal.go:1–9, 14–15; start.go:1–4, 55–56; status.go:1–2; stop.go:1–2; cycle.go:1–2; watch.go:1–2; spawn.go:1–2; session.go:1–2; handover.go:2–5; cli_test.go:1–3; start_test.go:1–2; watch_test.go:1–2; prime_integration_test.go:3–4; testmain_test.go:3–12; testmain_integration_test.go:3–7; smoke_cycle_test.go:3–15, 73–74, 240–241)
**Issue:** Same convention; the focus directive's named file headers (cli.go, refusal.go, start.go, status.go, testmain_integration_test.go, smoke_cycle_test.go) all break mid-sentence, e.g. testmain_integration_test.go "since / prime_integration_test.go spawns git".
**Fix:** Rejoin each sentence, keeping breaks only at independent-clause boundaries.

### [MEDIUM] Smoke test does not wait for the background-task turn to end

**Location:** internal/orchcli/smoke_cycle_test.go:184
**Issue:** Card 14 step 3 sends a turn that starts a background task "and waits for that turn to end".
The wait condition is `latestTranscript(...) != "" && smokeStatusIdle(...)`.
The first half is already true from the start prompt's Stop (line 179), and the second is true on any capture taken before Claude draws its running-turn hint, so the wait can pass on its first poll, before the turn has even begun.
The background-task turn's Stop names the same transcript, so `preCycleTranscript` does not prove the turn ended either.
**Fix:** Count the turn-end payloads in the events file before sending, and wait until the count grows past it.

### [MEDIUM] Smoke-test open-risk observations never gather their answers

**Location:** internal/orchcli/smoke_cycle_test.go:231
**Issue:** Card 14 requires observing whether the `SendMessage` address before and after `/clear` matches, read through `ListAgents` output.
Nothing ever asks the session to run `ListAgents`, so the logged pane cannot contain that output, and the log line itself concedes it is "not machine-checked".
The background-task observation reads the pane the moment the cycle settles.
The `sleep 40` may not have finished by then, so a `false` answer cannot tell "notification lost" from "not yet due".
**Fix:** Before and after the cycle, send a turn asking the session to report its own agent name from `ListAgents` on one marked line, wait for each turn end, and log both answers and whether they match.
Wait until the background task's deadline has passed, plus a margin, before reading the pane for `BGDONE`.

### [LOW] Package doc misstates the clearing timeout

**Location:** internal/orchengine/doc.go:82
**Issue:** "A phase that exceeds the handoff timeout ... aborts back to idle with LastAbortReason set."
`tickClearing` on timeout moves to `resuming` and sends the resume prompt (card 8, D10), without aborting.
Only `handoff-requested` and `resuming` abort on timeout.
**Fix:** State that a handoff or resume timeout, or a question during the handoff, aborts to idle, and that a clearing timeout moves on to resuming.

### [LOW] every-return-to-idle cursor persistence is not asserted for three of the returns

**Location:** internal/orchengine/watcher_test.go:461
**Issue:** Card 8's acceptance case "every return to idle → LastInjectionOffset persisted as the cursor read through" is asserted only in `TestWatcher_ResumeTurnEndTakesFirstPostCycleReading` and `TestWatcher_PreClearTurnEndNeverReadNorCyclesAgain`.
The handoff ask abort, the handoff timeout and the resume-render abort return to idle through the same `toIdle`, but no test pins their persisted offset.
**Fix:** Add a table test that drives each of those returns and asserts `LastInjectionOffset` equals the events read through.
