---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/04-spec-resume-session.md:14
    summary: Card 4 introduces the --resume launch edge but its Intent states neither what it lets through nor what bounds it
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/13-orch-live-smoke.md:36
    summary: The orchengine prompt-suggestion residual is rewritten in card 13, not in card 3 which changes the behaviour, so doc and behaviour land in different commits
  - id: F3
    severity: LOW
    location: _lyx/plan/07-orch-config-keys.md:29
    summary: Card 7's ImpactSummary says callers are unchanged, hiding that every present orch.yaml without the new keys fails to load until reconciled
  - id: F4
    severity: LOW
    location: _lyx/plan/13-orch-live-smoke.md:37
    summary: Card 13's stop-and-report rule for an unsubmitted --resume prompt does not carry the decision record's pre-decided fallback forward
  - id: F5
    severity: LOW
    location: _lyx/plan/13-orch-live-smoke.md:21
    summary: TestSmokeOrch_SoftCycle with soft_threshold_tokens 1 and a short soft_idle_s can fire the soft cycle on the start prompt's turn end before the background shell exists
  - id: F6
    severity: NIT
    location: _lyx/plan/11-orch-spec-bypass-agents.md:30
    summary: Card 11's ImpactSummary restates the Intent and does not state the removed function's caller count or that every orch-launched run still gets the Agent tool
  - id: F7
    severity: NIT
    location: _lyx/plan/12-orch-start-adopt.md:20
    summary: Card 12 names only starterFunc as the test fake to update, but fakeStarter in start_test.go also implements sessionStarter, and its ImpactSummary does not count the seam's implementers
---

### [MEDIUM] Card 4 states no bound for the new resume edge

**Location:** `_lyx/plan/04-spec-resume-session.md:14`

**Issue:**
Card 4 adds `Spec.ResumeSessionID`, a new routing edge: a run that sets it takes over an existing Claude session with `--resume` instead of minting a new one.
Its Intent states only the id-shape validation.
On its own, `Prepare` then launches any well-formed id with no transcript check and no live-holder check, so two processes can drive one session.
The guard that bounds this is in card 5 (`Runner.start` refuses a resume through an engine without `SessionResumer`) and card 6 (the transcript and registry check).
Card 4's own Intent never says so.
The rubric's attack-surface rule requires the card that introduces the edge to state what it lets through and what bounds it, so card 4 fails it even though the bound exists elsewhere in the plan.

**Fix:**
Add a `Bound:` paragraph to card 4's Intent.
It states that the engine resumes any well-formed id it is handed and checks nothing else.
Card 5's preflight in `Runner.start` bounds it: that preflight refuses a resume unless the engine implements `SessionResumer`, and card 6's check refuses a missing transcript or a live holder.
orch's `--adopt` is the only production caller.

### [MEDIUM] Prompt-suggestion residual is updated in the wrong card

**Location:** `_lyx/plan/13-orch-live-smoke.md:36`

**Issue:**
Card 3 switches prompt suggestions off in every generated settings file and rewrites `IdleSession`'s doc comment.
The orchengine package doc's first residual still says the idle probe never cycles a session showing Claude's greyed suggestion, and card 13 rewrites that residual, not card 3.
That splits a behaviour change from its doc across two commits, against the decision record's constraint that documentation lands in the same commit as the behaviour.
The residual's new text does not depend on the live run: it follows from card 3 alone.
Card 13 also has a stop path (an unsubmitted `--resume` prompt), so the stale residual could outlive the task's commits.

**Fix:**
Add `internal/orchengine/doc.go` to card 3's Edit list and have card 3 replace the residual with the settings fix.
Drop the residual sentence from card 13's Intent.

### [LOW] Card 7's ImpactSummary hides the config-load break

**Location:** `_lyx/plan/07-orch-config-keys.md:29`

**Issue:**
Card 7's own Intent says a present `orch.yaml` without the three new keys fails to load with the "run lyx config reconcile" error.
`configengine` raises exactly that error on missing keys (`internal/configengine/config.go:122`).
Every `lyx orch` verb on the hub's existing `orch.yaml` therefore fails until it is reconciled.
The ImpactSummary instead says "existing accessors and callers are unchanged", which leaves out the card's real blast radius.

**Fix:**
Rewrite the ImpactSummary to state the conclusion.
Every orch verb fails to load an existing `orch.yaml` until `lyx config reconcile` adds the three keys.
The in-repo fixtures that seed one (`config_test.go`, and the smoke test, which card 13 updates) need the keys.
No module outside orch reads the config.

### [LOW] Card 13's stop rule drops the decision record's fallback

**Location:** `_lyx/plan/13-orch-live-smoke.md:37`

**Issue:**
The decision record's open risk already decides the response if `claude --resume <id>` with a positional prompt does not submit the prompt.
That response is to resume without a prompt and send the one-line adopt stencil once startup clears.
Card 13 stops and reports, which is reasonable, since the fallback touches files outside its targets.
But its report names no way forward, so the pre-decided fallback could be argued over again.

**Fix:**
Keep the stop.
Have the report name the decision record's fallback as the follow-up fix: resume without a positional prompt in claudeengine's launch line and send the adopt stencil through the run once startup clears.
Have it record the observation in the orchengine Open risks.

### [LOW] Soft-cycle smoke test can trigger before its setup turn

**Location:** `_lyx/plan/13-orch-live-smoke.md:21`

**Issue:**
With `soft_threshold_tokens: 1`, every known reading qualifies, including the reading at the start prompt's own turn end.
Under a "short" `soft_idle_s` the watcher can send the soft handoff request before the test's background-shell turn reaches the session.
Then the handoff names no background shell and the assertion fails, or the test's turn is typed into a session that is mid-handoff.
The existing smoke test avoids this by putting the threshold out of reach and triggering manually.

**Fix:**
State in card 13 that `soft_idle_s` must be long enough to cover the test's own reaction to the start turn end (tens of seconds, not a few).
The background-shell turn's end then becomes the newest event before the start turn's quiet window closes.

### [NIT] Card 11's ImpactSummary restates its Intent

**Location:** `_lyx/plan/11-orch-spec-bypass-agents.md:30`

**Issue:**
Card 11 removes `orchShuttleConfig` and `TestOrchShuttleConfig_ClearsOnlyAgentDeny`.
Its ImpactSummary repeats that and adds only "no other module's runs or config change".
It does not state the removed function's reach: one caller, in `cli.go`'s `PersistentPreRunE`, and one test, both in orchcli.
It also does not state why the removal is safe: `orchSpec` is the only spec orch's runner ever starts (`start.go`'s single `StartSession` call), so every Claude run orch launches still gets the Agent tool through `AllowAgentTool`.

**Fix:**
Rewrite the ImpactSummary to state those conclusions.

### [NIT] Card 12 under-names the starter seam's implementers

**Location:** `_lyx/plan/12-orch-start-adopt.md:20`

**Issue:**
`sessionStarter` is implemented by `runnerSessionStarter` and by two test fakes in `start_test.go`: `fakeStarter` (line 30) and `starterFunc` (line 170).
Card 12 tells the implementer to update only `starterFunc`.
Its ImpactSummary says "one return value to its internal starter seam" without counting who implements it.

**Fix:**
Name both fakes in the Intent.
Have the ImpactSummary say that the seam has one production implementer and two test fakes, all in orchcli.

## Verdicts on the focus directives

- Card 11's removal of `orchShuttleConfig`: licensed.
  Decision 3 makes the allowance per run "whatever shuttle's `claude_deny_agent_tool` says" and keeps every other run on the config-driven deny.
  A config rewrite on orch's runner would strip that deny from any other run the runner started, so dropping it carries Decision 3 rather than adding scope.
  orch starts exactly one kind of run, `orchSpec`, and card 11 sets `AllowAgentTool` on it, so every Claude run orch launches still gets the Agent tool with `shuttle.yaml` loaded unmodified.
  Only the ImpactSummary wording is raised (F6).
- Card 4's bound: a finding (F1).
- Card 9 keeping `LastDeferral` and `CycleTrigger` through `ResetForFreshLaunch`: not a defect.
  `ResetForFreshLaunch` clears only per-phase fields and already keeps every history field (`CycleCount`, `LastAbortReason`, `LastContextTokens`, `LastHandoff`).
  Decision 5's "with `LastHandoff` kept" describes that existing function, not a whitelist.
  A kept `LastDeferral` cannot hold the soft trigger on a fresh session.
  The soft gate needs a turn end read after the relaunch at least `soft_idle_s` ago, and that window always closes later than one opened by a deferral read before the relaunch.
  `status` reporting the last deferral and last trigger across a relaunch matches Decision 6.
- Card 9's DEFER rules, checked one by one: the exact match on the trimmed final message, a written file beating DEFER, the deferral window measured from the read time, DEFER ignored in hard and requested cycles, and the hard-before-requested-before-soft order are all carried.
  The extras (re-stat on the DEFER tick, re-sending the soft stencil after a watcher restart) implement Decision 6's "written file wins" and its recorded trigger, and add no scope.
- Card 13 against Decision 7 and the acceptance criteria: every live item maps to a concrete step: bypass with no dialog, an Agent subagent, the idle probe, a soft cycle with a re-armed background shell, adopt keeping context and completing a cycle, `<shortname>:orch`, the loom `parent`, SendMessage reachability, and both recorded observations.
  The soft-cycle step's timing is raised (F5), and the stop rule's fit to the open-risk fallback is raised (F4).
- Documentation: the orchengine package doc is covered by cards 9, 11, 12 and 13, the shuttleengine package doc by cards 1, 2 and 4, and the `orch.yaml` template comments by card 7.
  The package doc's status entry lists no fields, so card 10's fields need no package-doc change, and card 10 updates the verb's `Long` text.
  The prompt-suggestion residual is assigned, but to the wrong card (F2).
- ImpactSummary on the Edit cards: 1, 2, 3, 4, 5, 9 and 10 state a conclusion (no existing caller or run changes, byte-identical output, or untouched paths).
  Cards 7, 11 and 12 are raised (F3, F6, F7).
