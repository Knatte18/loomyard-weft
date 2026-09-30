---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/08-orch-cycle-machine.md
    summary: A return to idle after an aborted handoff or a resume timeout leaves the pre-abort turn end and the pre-clear context reading live, so the watcher can re-cycle a freshly cleared session, against D7, D11 and the pre-clear acceptance case
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/14-orch-smoke.md
    summary: The smoke test seeds a low threshold and then asserts cycle_count settles at 1, but a threshold below the resumed session's own usage re-triggers a cycle on every turn end, including the start prompt's and the resume turn's
  - id: F3
    severity: LOW
    location: _lyx/plan/08-orch-cycle-machine.md
    summary: The restart test pair applies the landed/not-landed split to clearing too, which contradicts card 8's own unconditional re-clear rule for clearing, and no case pins the restart-in-clearing path D10 describes
  - id: F4
    severity: LOW
    location: _lyx/plan/09-orch-watch-loop.md
    summary: WatcherLive probes by briefly taking the watch lock, so a watcher spawned while status or start holds that probe lock exits as already_running and leaves no watcher
  - id: F5
    severity: LOW
    location: _lyx/plan/10-orch-cli-skeleton.md
    summary: The orch-agent-tool override is set inline in PersistentPreRunE and no test pins it, so a regression silently restores the Agent deny in the orchestrator session
  - id: F6
    severity: LOW
    location: _lyx/plan/04-orch-config.md
    summary: The ImpactSummary omits fabric clone's CloneAndWire reconcile, which will materialise and commit _lyx/orch.yaml into every new hub's weft, hubforge fixtures included
  - id: F7
    severity: NIT
    location: _lyx/plan/12-orch-start-verb.md
    summary: The ImpactSummary says no code outside the module changes while the card edits cmd/lyx/helptree_test.go
---

### [BLOCKING] A return to idle does not retire the turn ends and reading the cycle already saw

**Location:** `_lyx/plan/08-orch-cycle-machine.md` (the `idle`, `handoff-requested` and `resuming` paragraphs), with the `events-cursor` decision in `00-overview.md` and `LastInjectionOffset` in `06-orch-state.md`.

**Issue:**
Card 8 advances `LastInjectionOffset` only when `resuming` completes, and it never touches the in-memory newest event or the persisted reading on any other return to `idle`.
Three paths follow:

1. Resume timeout, same watcher.
   The trigger reading is persisted as `LastContextTokens` before the cycle and is taken only from turn ends read in `idle`, so the handoff turn end (read in `handoff-requested`) never refreshes it.
   If the resume prompt never produced a turn end (Send swallowed, resume timed out), the watcher is back in `idle` with the handoff turn end as its newest event, the grace long expired, a cleared session whose empty box passes `SessionIdle`, and a stale pre-clear reading over threshold.
   The next tick injects a second handoff into the blank session.
   That handoff becomes `LastHandoff`, so D6's resume source now points at a near-empty file instead of the real one.
2. Resume timeout, restarted watcher.
   The `events-cursor` decision starts an idle watcher at `LastInjectionOffset`, which still names the previous completed cycle (or 0).
   The watcher re-reads the pre-clear turn ends in `idle`, including the handoff turn end, and takes readings from the transcript each names: the old, pre-clear one.
   This is exactly the acceptance case "pre-clear turn end after `/clear` → not evaluated for usage, no second cycle", reached through the abort path instead of the success path.
3. Aborted handoff whose turn end was already read.
   After a timeout where the handoff turn ended but no file appeared, the newest event is that turn end, so the very next idle tick can re-inject at once.
   D11 says the next attempt waits for the next qualifying idle turn end, and card 8 repeats that sentence without a mechanism behind it.

D7's "only a turn end after the watcher's last injection is evaluated" is met only on the success path.
None of card 8's test cases drives a tick after an abort, so the list does not catch this.

**Fix:**
On every return to `idle` (resume completion, resume timeout, handoff abort), persist `LastInjectionOffset` as the cursor the watcher has read through and forget the in-memory newest event, so only a turn end read after the return can qualify.
On a return after `/clear` without a resume turn end (resume timeout), also mark the reading unknown, since it predates the clear.
Restate `LastInjectionOffset` in card 6 and the `events-cursor` decision to match.
Add card 8 cases: ticks after a resume timeout with an idle pane and no new turn end inject nothing; a handoff abort whose turn end was already read does not retry until a later turn end; and a watcher restarted in `idle` after a resume timeout does not re-read the pre-clear handoff turn end.

### [MEDIUM] The smoke test's low threshold cannot settle at one cycle

**Location:** `_lyx/plan/14-orch-smoke.md`, steps 1, 3 and 5.

**Issue:**
Step 1 seeds "a low `threshold_tokens`" with a short idle grace, and step 5 waits for `cycle_count` 1 and `phase` `idle`.
A fresh Claude Code session already holds its system prompt and tool definitions, so any threshold low enough to matter is crossed by the start prompt's own turn end.
That fires a threshold cycle before step 3's background task exists.
Every later turn end crosses it too, including the resume turn's, so the session cycles again and again, and `cycle_count` passes 1 at a moment the test does not control.
The background-task and resume-prompt assertions then observe whichever cycle happened to be last.

**Fix:**
Seed `threshold_tokens` far above any reading the smoke session can reach, so the manual `lyx orch cycle` is the one trigger.
State the reason in the card: a threshold under the resumed session's own usage re-triggers on every turn end, and the threshold-triggered path is covered by card 8's unit cases.

### [LOW] The restart tests contradict the clearing restart rule

**Location:** `_lyx/plan/08-orch-cycle-machine.md`, the last three test bullets and the restart-rule paragraph.

**Issue:**
The restart rule for `clearing` is unconditional: re-call `ClearSession`, then wait, which follows D10 as written.
The test list then asks, for "each non-idle phase", that a landed injection produce no duplicate send and an unlanded one exactly one re-send.
For `clearing` the first half contradicts the rule, and no case asserts the restart-in-`clearing` behaviour itself.
On the focus question: the first `SessionIdle` probe after `ClearSession` passing against a pane where `/clear` has not yet taken effect is the risk D10's rationale accepts ("neither phase needs proof that `/clear` landed"), so card 8 implements D10's `clearing` text faithfully.
The gap is the test list only.

**Fix:**
Scope the landed/not-landed pair to `handoff-requested` and `resuming`, and add a case: a restart in `clearing` calls `ClearSession` exactly once, then sends the resume prompt once the probe passes.

### [LOW] The watcher-liveness probe can starve a starting watcher

**Location:** `_lyx/plan/09-orch-watch-loop.md`, `WatcherLive` and `Run`.

**Issue:**
`WatcherLive` answers by taking the watch lock with `TryAcquireWriteLock` and releasing it.
`Run` treats a failed `TryAcquireWriteLock` as `ErrWatcherRunning`, and card 11 reports that as a success envelope and exits.
A watcher spawned while `lyx orch status` holds the probe lock therefore exits, and no watcher runs until the next `start`.
The window is short, but `status` is the verb an operator polls.

**Fix:**
Have `Run` retry the lock a small, capped number of times over a short interval before concluding `ErrWatcherRunning`, and add a `watchloop_test.go` case where the lock is released between attempts.

### [LOW] The Agent-tool override has no test

**Location:** `_lyx/plan/10-orch-cli-skeleton.md`, resolution step 3.

**Issue:**
On the focus question: D16 makes the strand-hosted session "an ordinary local Claude Code session", and the Goal is hosting the operator's own orchestrator, which uses the Agent tool.
Shuttle's default denies that tool in every run, so the override is what makes D16 true; it is licensed, not new scope.
It is set inline in `PersistentPreRunE`, though, which `cli_test.go` never runs, and card 12's spec test cannot see it because the deny lives in the runner's config, not the spec.
A regression would silently strip the orchestrator's Agent tool.

**Fix:**
Extract the override into a pure `orchShuttleConfig(cfg shuttleengine.Config) shuttleengine.Config` in `cli.go`, and add a `cli_test.go` case that it clears `ClaudeDenyAgentTool` and leaves every other field alone.

### [LOW] Card 4's ImpactSummary misses the clone path

**Location:** `_lyx/plan/04-orch-config.md`, `ImpactSummary`.

**Issue:**
`configreg.Modules()` feeds `configsync.ReconcileAll`, and `fabriccli`'s `CloneAndWire` runs that with apply on every clone (`internal/fabriccli/clone.go:100`).
So the new entry makes every new hub, and every `hubforge` integration fixture, carry a committed `_lyx/orch.yaml`.
`fabriccli`'s clone-config commit test derives its expected set from the registry, so it keeps passing, but the summary's "reaches `lyx config`'s listing, reconcile and menu" stops short of that.

**Fix:**
Name the clone-time reconcile and its committed `_lyx/orch.yaml`, and note that the clone-config commit test derives its set from the registry.

### [NIT] Card 12's ImpactSummary contradicts its own Edit list

**Location:** `_lyx/plan/12-orch-start-verb.md`, `ImpactSummary`.

**Issue:**
It ends "no code outside the module changes", while the card edits `cmd/lyx/helptree_test.go`.

**Fix:**
Say that outside the module only `cmd/lyx/helptree_test.go` changes, gaining `start` in the `orch` row.

## Focus verdicts

- **D13/D7 versus `session-cycler-capability`:** not a defect.
  D13's rationale is the Shuttle Provider-Seam Invariant: `/clear` is key grammar and belongs behind the seam, in `claudeengine`.
  An optional capability interface declared in `shuttleengine` and implemented by `*Claude` keeps it there, and D7's "shuttleengine and orchengine see only a provider-invariant token count" still holds.
  The overview states the departure from the literal "new `Engine` method" and gives a grounded reason: eleven `Engine` fakes across five packages (`ModelSwitchSequence` implementers in `websterengine`, `webstercli`, `loomcli`, `shuttlecli`, `shuttleengine`) would gain methods with no behaviour.
- **`orch-agent-tool`:** licensed by D16 and the Goal; see F5 for the missing test.
- **`LastInjectionOffset` and the cursor:** a defect; see F1.
- **Card 8's `clearing` phase:** implements D10 as written; see F3 for the test list.
- **ImpactSummary claims:** cards 1, 5, 10 and 11 hold against the code; cards 4 and 12 are F6 and F7.
  Card 14's summary is accurate.
- **Acceptance coverage:** every bullet maps to a card.
  Start/second-start/refusals: card 12's `start_test.go` and card 10/12's `prime_integration_test.go`.
  Corpse removal, last-handoff resume and `--handoff`: card 12.
  `status` fields: card 10.
  `cycle`: card 8's manual-request case and card 14.
  State-machine cases: card 8, except "fresh launch over a non-idle phase" (cards 6 and 12) and "aborted handoff never chosen" (card 7).
  Transcript fixtures: card 2; pane fixtures: card 3.
  Smoke: card 14, flawed per F2.
  Seam tests: the verify section runs `cmd/lyx` and `contracts/stencils`.
  Docs and `CONSTRAINTS.md`: card 13.
  The pre-clear case is carried only on the success path, which is F1.
- **Granularity of cards 3, 10, 11, 12:** no finding.
  Card 3's two methods together complete one interface that its compile-time assertion pins.
  Cards 11 and 12 pair a verb with unexported supporting types (`runnerSession`, `spawnWatcher`, `decideHandover`) whose only consumer is that verb, which the rubric places in the same card.
  Card 10's three verbs are thin readers over functions cards 6 and 9 already test, so the card's weight is the resolution wiring, one reviewable unit.
