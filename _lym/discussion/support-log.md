# Support log — creel-hook-post

## Interview

The session ran autonomously with no advisors; every turn below is the writer's own research and pick.

### Turn 1 — Task and context

Read the board entry `creel-hook-post`, the board list (no `Decision for creel-hook-post:` note), the two research reports the entry cites, the landed commit `3422ceac` (PostToolUse hook, flags, `ClaimAtBoundary`), `creelengine`'s package doc, `hookread.go`, `claudeengine`'s settings, launch line and events parser.
Touching notes: `codex-engine` (no engine yet, so Codex stays out), `creel-agents` (stage 3, separate).

### Turn 2 — Facts researched

- Claude Code hooks docs: `asyncRewake` timeout is enforced (default 600 s); `Stop` does not run on a user interrupt; 8-consecutive-continuation cap; `agent_id` present in subagents and in-process teammates; `SubagentStop` fires for subagents.
- Claude Code 2.1.312 bundle: the rewake summary shown to the operator is the stdout `rewakeSummary` only for official-marketplace plugins, else the `@internal` hook-entry `rewakeSummary`, else `Stop hook feedback`; the hook-input builder sets `agent_id` from the tool-use context.
- Conclusion: a settings hook cannot show rewake text to the operator, so the entry's "make the wake text visible" is met by a bell wake and delivery through the visible Stop block (D4).

### Turn 3 — Interaction found in code

The `Stop` recording hook logs every turn end to `events.jsonl`, and `ParseEvents` reads it as `EventStop`; a parallel creel Stop block would leave a turn-end record for a turn that goes on.
Pick: merge creel's Stop delivery into the recording command and skip the record on a block (D2).

### Turn 4 — Idle sessions with no waiter

A user interrupt fires no `Stop`, so after `post --interrupt` no new waiter starts, and waiter survival across turns is undocumented.
Pick: a typed fixed-text bell only when no waiter is live (D5), raised under Open risks since the entry says the pane path goes.

### Turn 5 — Remaining picks

Quoting every sender name (D8), `--interrupt --reply` refused (D7), sentence on every session (D9), 86400 s waiter timeout (D4).

### Turn 6 — Lens review

All seven forks returned a report.
Extra research for the fixes: hooks docs (10,000-character cap per output field, matching hooks run in parallel), `internal/shell` `Quote` forms, `claudeengine`'s notice guard.

- interactions 1 (BLOCKING, ring then typed bell): fixed — one wake per claimable set; a ring counts; D5 only after a 5 s no-waiter grace, an unanswered ring or the cap.
- interactions 2 (cap discards claimed mail, no turn end): fixed — creel's own continuation count, cap 6, reset by a `PostToolUse` claim or a `Stop` with `stop_hook_active` false (D2).
- interactions 3 (rewake counts toward the cap): fixed — rings are counted with blocks (D2).
- existing-paths 1, walkthrough 1, bound-breaker 2 (stdin read twice; flag gate would skip recording): fixed — payload read once and passed to both steps; flag gates only the creel step (D2).
- existing-paths 2, bound-breaker 5 (daemon restart drops waiters): fixed — waiter reconnects and re-registers, never exits 2 on a drop (D4).
- existing-paths 3 (wake turn has no `UserPromptSubmit`): fixed — bound stated in D4, relying on the guarded send's busy reading.
- existing-paths 4 (unprompted turn end recorded): fixed — bound stated in D4.
- existing-paths 5 (old records fold): rejected — already covered by the `PATTERN-tolerant-records` constraint, as the lens says.
- walkthrough 2: duplicate of interactions 2, fixed there.
- walkthrough 3 (second ring after the mail turn): fixed — one wake per claimable set.
- walkthrough 4 (bell leads to `lym creel read`): fixed — bell text says to end the turn and not run `lym creel read`.
- walkthrough 5 (bell before the waiter connects): fixed — 5 s grace (D5).
- walkthrough 6 (how the verb learns the event): fixed — from `hook_event_name` (D1).
- bound-breaker 1 (`"` in the sentence against the notice guard): fixed — the sentence is its own constant refusing only line breaks; `Quote` facts recorded.
- bound-breaker 3: duplicate of interactions 1, fixed there.
- bound-breaker 4 (interrupt sent, then post refused): fixed — validate before the key (D7).
- bound-breaker 6 (cap bound unproven): fixed — creel's count stays below Claude Code's cap, and D5 takes over at it.
- bound-breaker 7 (orphan waiter blocks the bell): fixed — an unanswered ring hands the set to D5 after 30 s.
- bound-breaker 8 (hook output size): fixed — researched; 10,000-character cap per field, so 8 KiB in each of `reason` and `systemMessage` fits; constraint added.
- bound-breaker 9 (where a stale bell lands): fixed — bound reworded.
- goal-scope 1 and 2 (`decision`): kept as parent questions under Open risks, phrased as questions.
- goal-scope 3 (LLM-tier qualifier): fixed — one line covering every real-Claude criterion.
- goal-scope 4 (quoting beyond strand names): rejected — one line shape for every sender is harmless and recorded as an auto-pick.
- board-fit 1 and 2 (`creel-agents` rests on the reminder; reply-discipline overlap): fixed — note for the plan writer; the note's body stays the operator's.
- board-fit 3 (a new engine needs its own hooks): fixed — constraint added.
- leanness 1 (Notes listing read code): fixed — bullets cut.
- leanness 2 (rewake internals): fixed — constraint trimmed.
- leanness 3 (timer fact twice): fixed — folded into D4.
- leanness 4 (Open risk restating D5): fixed — dropped.
- leanness 5 (parent risk not a question): fixed.
- leanness 6 (Scope In repeats decisions): fixed — one sentence.
- leanness other (parallel-hooks citation): fixed — constraint added.

## Rejected alternatives

- Waiter carries the mail in its stderr: the model gets it in one turn, but the operator sees only `Stop hook feedback`, against the entry's visibility goal.
- `rewakeSummary` on the hook entry: `@internal` in the 2.1.312 schema and static text, so it cannot show the message and may break settings validation when removed.
- A separate creel `Stop` hook beside the recording hook: Claude Code runs them side by side, so the recording would log a turn end that the block cancels.
- `Stop` block with `hookSpecificOutput.additionalContext`: labelled `Stop hook feedback` with no text shown.
- Relying on the waiter alone for interrupted or long-idle sessions: no `Stop` after a user interrupt, and the waiter ends at its timeout.
- A waiter timeout near the JavaScript timer ceiling: a value above 2^31−1 ms fires at once, and a day is enough with D5 behind it.
- Filtering subagents on `agent_type`: `--agent` sets it in the main thread too.
- Typing the bell into hand-added strands: they have no hooks, so the bell would promise a delivery that never comes.
- Gating D2 on `stop_hook_active` alone: a rewake turn carries it as true, so the woken turn's Stop would never deliver.
- Changing the D9 sentence to avoid `"`: the entry gives it verbatim, and both `Quote` forms carry `"`.
- A waiter that exits on a dropped connection: every deploy would leave idle strands to the typed bell.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): How does the idle wake become visible? — The wake carries a fixed bell and the next boundary's block carries the mail (D4); the entry's "stop and ask the parent when no way" is not needed since a way exists, and the trade is raised under Open risks.
- Q2 (auto-pick): How does a blocked Stop interact with turn-end recording? — One merged command; a block records no turn end (D2).
- Q3 (auto-pick): What reaches an idle session with no live waiter? — A typed fixed bell once per new mail set, shuttle-launched strands only (D5); raised under Open risks.
- Q4 (auto-pick): Waiter timeout? — 86400 s.
- Q5 (auto-pick): Does a draft hold the wake? — No; the spike shows the wake leaves a draft untouched.
- Q6 (auto-pick): Which names does D8 quote? — Every sender name in `MessageLine` and `read`'s output.
- Q7 (auto-pick): `--interrupt` with `--reply`? — Refused with the way forward.
- Q8 (auto-pick): Which sessions get the D9 sentence? — Every Claude session shuttle launches, as the entry says.
- Q10 (auto-pick): How does D2 stay clear of Claude Code's continuation cap? — Creel's own count of blocks and rings, cap 6, reset by a `PostToolUse` claim or a `Stop` with `stop_hook_active` false.
- Q11 (auto-pick): How long before an unanswered ring or a missing waiter falls back to the bell? — 30 s and 5 s.
- Q12 (resolved by research): Does an 8 KiB claim fit the hook output? — Yes: 10,000 characters per field (hooks docs, JSON output).
- Q9 (resolved by research): Does a fork's hook input carry `agent_id`? — Yes by the docs and the 2.1.312 hook-input builder; the LLM-tier test pins it.
