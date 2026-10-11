# Support log: shuttle-hook-first

## Interview

Autonomous mode: no operator answered; every pick below is the writer's, recorded in the question ledger.

Turn 1 — read the board entry `shuttle-hook-first` and every entry touching it: `parent-review-wait` (depends on this task), `shuttle-color-verified-send`, `lyx-halt`, `extra-agent-strand`, `creel`.
None plans a change to the fold, the hook set or the startup probe; `shuttle-startup-ready` has landed at 567170457.

Turn 2 — explored the fold (`sessionstate.go`), the signal parser (`claudeengine/signals.go`), the settings builder, `readiness.go`, `busyReading`, `awaitStartup`, `checkLivenessTick`, the registry prober, `holdsPendingToolCall`, and the draft refusals in `loomcli` and `battenshed`.
Read this run's own `events.jsonl`: real `UserPromptSubmit` and `Stop` payloads, with `session_id`, `transcript_path`, `permission_mode` and `background_tasks`; no `SessionStart` line because none is installed without a compaction command.
This session's own context shows a plugin `SessionStart:startup` hook ran before the first prompt.

Turn 3 — settled the fold rule.
A `SessionStart` keyed on the hook name alone would misread an automatic compaction inside a running turn as idle, and a rule "only before any turn signal" would leave a resume after a mid-turn crash busy forever.
So the rule keys on the source, with `compact` and unknown sources failing safe.

Turn 4 — settled startup: require both the hook line and the pane, leave gate dismissal untouched.

Turn 5 — settled death: positive evidence only, and the `SessionEnd` path vetoed by a live registry entry.

Turn 6 — settled #554: clear once, then read a persisting character as residue.

Turn 7 — settled the fork transcript tail rule.

Turn 8 — lens review by six forks; every fork returned a report.

- premises, goal-scope, scenarios — D9 assumed one assistant entry per batch; Claude writes each parallel call as its own entry under one `message.id`: fixed, D9 groups by `message.id`, with a parallel-batch criterion.
- goal-scope, premises — D8 did not reach the pane-fallback reading that the Open risks claimed it covered: fixed, D8 applies on both readings and quotes the text on the fallback too.
- scenarios — D7's registry evidence would fail a resumed launch on the crashed launch's stale entry: fixed, registry death counts only once this wait has read the session id alive, with a criterion.
- scenarios — D8's residue reading after a clear could fire on a stale capture: fixed, a redraw settle before the re-read, and the bound admits a real character that survives.
- scenarios — D8's bound understated the loss; C-u also wipes text typed between the reading and the clear: fixed in D8's bound.
- scenarios — D4 did not say whether an attached not-yet-started run applies it, and an earlier launch's line satisfies it: fixed, the rule lives in `checkLivenessTick`, and the stale-line case is stated as a bound.
- scenarios — missing criteria for an unreadable events file, a clear sequence, `/exit`, and `compact` recorded without a compaction command: fixed.
- scenarios — D6 had no wait bound: fixed, the existing landing window.
- scenarios — D7's cadence wording disagreed with Constraints: fixed.
- scenarios — a lingering ask in an autonomous run may show in batten's or the orch's view: fixed in Open risks.
- board-fit — a launch-line first turn (`shuttle-color-verified-send`) could land `UserPromptSubmit` before `SessionStart(startup)`: fixed, a `startup` start never overrides a running turn, with a criterion and a Constraints line.
- pattern-fit — D8's clear is a key delivery into a box not holding the sent text, which the background's list must name: fixed in Constraints.
- pattern-fit — D7's interaction with the transient mark was unstated: fixed, the same `OutcomeDied` as a dead pane, no new marking.
- pattern-fit — the stray-character judgment's side of the seam was unstated: fixed, `claudeengine` owns it.
- pattern-fit — D4's and D7's extra reads per probe and tick: fixed in Constraints.
- pattern-fit — `docs/overview.md` needs no change: noted, nothing to fix.
- premises — D4's rationale stated the trust proof as fact while Q2 called it unknown: fixed, conditional on the corpus, with a capture criterion and an Open risk.
- premises — `lyx loom resume` already quotes the draft; only batten's notice lacks it: fixed in D8.
- premises — D6 belongs in `sendVerified`'s no-turn branch: fixed in D6.
- premises — the registry assumption was unverified: fixed, D1 records the registry after `/exit` and a kill.
- goal-scope, leanness — the stop-and-escalate condition sat in Open risks, and D3 shares the dependency: fixed, moved into D1 naming D3 and D4.
- goal-scope — D4 declines board step 3 only in the support log: fixed, the Scope Out bullet names it.
- goal-scope — `StopFailure` coverage unstated: fixed in D1.
- leanness — Goal's "Concretely" paragraph and Scope's In list restated the Decisions: fixed, both cut.
- leanness — D7's shadow-log sentence, the `PATTERN-sole-parsers` line, assumption evidence, the `agent_needs_input` assumption and two Notes bullets were redundant: fixed, cut.
- leanness — Scope Out bullets carried rationales also in Rejected alternatives: fixed, rationales cut.
- leanness — Q10 claimed board backing the entry does not give: fixed.
- board-fit — `parent-review-wait` depends on this task without the record saying what it needs: rejected, that entry consumes D5 and D7 as built and asks nothing of this plan.
- board-fit — `hold-deferral-end-on-held` and `extra-agent-strand` could use D5, D7 and `idle-fresh`: rejected, compatible and no design change; their own entries take it up.
- board-fit — `creel`'s hand-added pane idle check matches the kept pane path: no change needed.

## Rejected alternatives

- **Stop sending gate keys once `SessionStart` lands** (board step 3).
  The classifier cannot tell the trust gate from the bypass gate, and whether the bypass dialog comes before or after `SessionStart` is unknown; a bypass dialog drawn after it would then wedge the start.
  The rendered-gate evidence rules already keep keys out of a healthy pane.
- **Fold keyed on the hook name alone** ("a `SessionStart` with no later turn start reads idle").
  An automatic compaction inside a running turn writes `SessionStart(compact)` after the turn start, and the turn would read idle.
- **Reuse `idle-stalled` or `idle-done` for a fresh session.**
  `idle-stalled` means stopped without finishing and `idle-done` means the outputs exist; batten and the orch watcher would misreport a fresh session.
- **Run the dead-reading switch in the shadow log first, then switch.**
  A phase split inside one task; the evidence is positive-only and the shadow log keeps running, so it gives no extra safety.
- **Treat a lone stray character as an empty box, like `%`.**
  The paste would submit it at the head of the message whenever the character is real.
  D8's clear first makes the delivered text exact in every case but a real character surviving the clear and the settle.
- **Leave a lone stray character a draft and only quote it.**
  #554 stays: the run parks until the operator attaches.
- **`idle_prompt` as positive proof of an empty box.**
  It fires about 60 s late and no hook reports typing after it, so the box read is still needed.
- **reed's name repair through shuttle's readiness.**
  reed sits below shuttle and is provider-blind; the repair would need a dependency inversion for one check.
- **`PostToolUse` to clear an ask after a granted permission.**
  A hook on every tool call grows the events file for a display-only gain.
- **Stop the backward fork transcript read at the first tool result or pending call** (the board's wording).
  Reading backward, the first `tool_result` can belong to the last call of a parallel batch whose earlier call is still pending.
- **Registry evidence regardless of age** in D7.
  A crashed launch's entry would fail a resumed launch of the same session before its own `SessionStart(resume)` lands.
- **Registry evidence aged against the newest `SessionStart` line, or against the launch's events offset.**
  A process starts before its own line, so the age test voids every entry; and once a resumed launch's line lands, the crashed launch's dead entry still reads dead until the new process registers, so an offset boundary fails the resume too.
  Requiring this wait to have read the id alive first rules out both.
- **Count a process-ending `SessionEnd` once this wait read its session id alive**, beside the first-tick size rule in D7.
  The read keys on the id, not the process: a resumed launch sharing the id makes an earlier launch's line count, and one unproven registry tick before `SessionStart(resume)` lands ends the resume died.
  Dropping it costs only a `SessionEnd` written just before the first tick, which then waits on registry death or the deadline.
- **Fold adopts D7's registry veto of a process-ending `SessionEnd`.**
  The session announced its end, so holding sends is right; only the run's end waits for the registry.
- **Map `agent_needs_input` to an ask.**
  It would override an `idle-done` reading of a finished run.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Does `SessionStart(startup)` land before the bypass dialog and before the box is drawn? — Unknown; the design does not depend on it: startup requires both the line and the pane (D4), and gate dismissal is unchanged. The corpus records the order.
- Q2 (auto-pick): Does the hold of settings-file hooks until trust apply to the `--settings` file? — Unknown; a `SessionStart` line proves trust passed only if the corpus shows the hold applies (D4's rationale), and without it D4 still requires the pane's ready reading. A line that never lands on its own is D1's blocker for D4.
- Q3 (auto-pick): Should `Run.Wait` end a run on `dead`, and on which evidence? — Yes, on a process-ending `SessionEnd` with no live registry entry, or a registry-proven dead pid; never on unproven (D7).
- Q4 (auto-pick): Which newer `Notification` types mean asking? — `elicitation_url_dialog` asks; `agent_needs_input` and `elicitation_response` yield nothing (D5).
- Q5 (auto-pick): `idle_prompt` as proof of an empty box? — No; it stays the contradiction marker.
- Q6 (auto-pick): reed name repair through shuttle? — No; stays pane-only.
- Q7 (auto-pick): Does a resumed session keep its `session_id`? — The liveness lookup takes the id from the newest signal naming one, so the `SessionStart(resume)` line points it at the live process either way; the corpus records the id.
- Q8 (auto-pick): #554 rule: treat as empty or clear? — Clear once behind the full guard, settle, then read a persisting character as residue (D8).
- Q11 (auto-pick): How does the implementer escalate if the corpus contradicts D4? — A blocker in its batch report, building every Decision but D4; D3's `startup` rule is built either way, since it moves nothing when no `startup` line lands (D1).
- Q12 (auto-pick): Which Claude behaviours does the design assume without proof? — That a `SessionStart(startup)` line lands from the `--settings` file with no turn typed, and that Claude removes its registry entry on a normal exit and leaves it on a crash. D1 records both; the first is D4's blocker, and D7 needs positive evidence either way.
- Q13 (auto-pick): Does `shuttle-color-verified-send`'s launch-line first turn break D3? — No; D3's `startup` bound holds for either hook order, so that entry needs nothing from this plan.
- Q9 (auto-pick): Who runs the `llm`-tier capture? — The implementer, through `lyx gate test --tags llm`, which the raw-go deny permits.
- Q10 (auto-pick): Should `requireReadyAgentPane` move to hooks? — No; it guards an explicit key delivery against a gate dialog, which no hook reports.
