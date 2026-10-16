---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-start-continue.md (Intent, the Bound sentence); _lyx/plan/02-watcher-records-session-id.md (Intent, the Watcher.recordSessionID bullet)
    summary: The bound on what --continue can resume drops D3's stated reliance on the provider naming the parent session id in subagent and fork hook lines, and card 2's new watcher write that feeds --continue states no bound on which id it accepts
    fix: In 03-start-continue.md, extend the Bound sentence after "the watcher (read from the bound strand's own shuttle run)" with "; the watcher's half rests on the provider naming the parent session id in a subagent's or fork's hook lines, since the orch strand runs with ForkSubagents and the claude engine takes each hook line's session_id as given, so a provider that names the subagent's own id there makes --continue resume the subagent's session". In 02-watcher-records-session-id.md, add to the Watcher.recordSessionID bullet that it accepts whatever id the newest id-naming signal of the strand's own run names, bounded to that run's events file, with the same reliance; and add to the card's internal/orchengine/doc.go paragraph that its `# Open risks` section gains this risk. No target changes.
  - id: F2
    severity: LOW
    class: scope
    location: _lyx/plan/04-status-session-id.md (Edit list and Tests, the TestStop bullet)
    summary: The TestStop extension pins that stop keeps the recorded session id, which card 3's TestStart_AdoptLaunchesResumeSpec row continue after stop resumes the recorded id already pins
    fix: In 04-status-session-id.md, drop `internal/orchcli#TestStop` from the Edit list and drop the TestStop test bullet; reword the Intent's stop bullet to end with "; TestStart_AdoptLaunchesResumeSpec's continue after stop resumes the recorded id row (card 3) pins it", drop "and the record outlives lyx orch stop (D4)" from the Intent's first line, and end the ImpactSummary with "stop's behavior is unchanged". Checked with lyx loom validate-plan --overlay against the {} baseline; no new finding.
---

### [MEDIUM] The --continue bound omits the provider reliance D3 records

**Location:** `_lyx/plan/03-start-continue.md`, the Intent's `Bound:` sentence; `_lyx/plan/02-watcher-records-session-id.md`, the `Watcher.recordSessionID` bullet.

**Issue:** D3's bound says `--continue` resumes only an id orch recorded for its own strand.
It also says the watcher's half "rests on the provider naming the parent session id in a subagent's or fork's hook lines, as Open risks records".
Card 3's `Bound:` keeps the first half and drops the reliance.
It then claims flatly that `--continue` "takes over no session orch did not launch, adopt or bind".
The code shows the reliance is real.
`orchSpec` sets `AllowAgentTool` and `ForkSubagents` on the orch strand.
`internal/shuttleengine/claudeengine/signals.go:124` takes each hook line's `session_id` unfiltered.
Card 1's `newestSessionID` then picks the newest id-naming signal of any kind.
The rubric's attack-surface bullet requires the card that introduces the routing edge to state what it lets through and what bounds it.
That card is card 2, where the watcher writes the id `--continue` later resumes.
Card 2 states the strand-checked save but nothing about which ids the reading can carry.
The executor writes this Intent into `doc.go` and comments, so the dropped hedge would reach the package docs as an unconditional guarantee.

### [LOW] TestStop extension duplicates card 3's stop-then-continue row

**Location:** `_lyx/plan/04-status-session-id.md`, the Edit list entry `internal/orchcli#TestStop` and its Tests bullet.

**Issue:** Card 3's row "continue after stop resumes the recorded id" seeds strand `g1` with `adoptTestSessionID`, runs `stopCmd`, then runs `--continue`.
It asserts that the resume id and the saved id both equal `adoptTestSessionID`.
If `stop` dropped the id, `--continue` would refuse on the no-id guard and that row would fail.
So the stop-keeps-the-id behavior is already pinned at the public surface.
Card 4's extra assertion in `TestStop` covers no further behavior.
`stop` (`internal/orchcli/stop.go`) writes no state on any path, so its tracked and untracked rows exercise the same non-write.
`PATTERN-test-economy` counts a redundant assertion as a defect.
The lens review (`lens-review.md`, the criteria-coverage bullet "no row runs stop then --continue in sequence") added card 3's row but left card 4's TestStop extension in place.

## Rejected

- Card order across 1–4: card 2 reads `RunSessionState.SessionID` (card 1), card 3 reads `Run.SessionID` (card 1) and `State.SessionID` (card 2), and card 4 reads `State.SessionID` (card 2); no card uses a symbol a later card introduces.
- Refusal reachability in card 3: each of the exclusive-with-`--adopt`, exclusive-with-`--handoff`, live-strand and no-recorded-id refusals has a named input and a `TestStart_Refusals` row, and the launch-error ways forward has a `TestStart_StarterErrorFailsAndSavesNoState` row; the no-id check sits before `RemoveStrand` in `startCmd`'s `StartRelaunch` branch, and `DecideStart` returns `StartRelaunch` for the dead `corpse` input, so the row reaches it.
- Attack surface of `--continue` and the adopt-branch resume: card 3 states the bound (only `st.SessionID`, its two writers, shuttle's resume check unchanged) apart from the gap in F1; the replaced-id hint is envelope text and lets nothing through.
- Member glyphs and re-sign arrows: every Edit card names members; the re-signed `runnerSessionStarter.StartSession`, `fakeStarter.StartSession` and `starterFunc.StartSession` carry arrows, `sessionStarter` is an interface and `starterFunc` a type, and no other listed member's signature changes (`readSessionFacts`, `readRunSessionState`, `Watcher.tick`, `adoptStrand` keep theirs).
- No other start-seam caller: `StartSession`, `starterFunc` and `fakeStarter` appear only in `internal/orchcli/start.go` and `start_test.go`, including under the `llm` and `integration` tags.
- `TestWatcher_LogsDisagreementBetweenIdleProbeAndSessionState` unchanged: with the per-tick read and the probe each consuming one `sessionSinces` entry per tick, the script `{t0, t0, t0+1m}` logs on ticks 1 and 2 only, matching `wantWarn: 2`, as card 2 claims.
- Test budget: `internal/orchengine` holds 144 top-level tests against its row of 144, so card 2's single new test needs the raise it lists; `internal/orchcli` (28 of 28) and `internal/shuttleengine` gain none.
- Verify: `## verify:` runs `./...` in the template's order with `go test -tags integration ./...`, covering `internal/orchcli`'s integration-tagged tests.
- Fidelity: D1–D6, the Scope In bullets, the Constraints and the acceptance criteria each map to a card; no card touches this task's board entry, and the no-parser `SessionID` fallback in card 1 is licensed by D4's "else the run record's".
