# Decision record — creel-hook-post

## Goal

Creel delivers every post to a Claude strand through hooks shuttle renders into the session's `settings.json`, as fast as possible, in full, visibly to the operator, and acted on by the receiver.
A scope change from the orch reaches a working agent at its next tool call, never only at its turn end.
No message body is ever typed into a pane.

## Scope

In: D1 to D11 below, covering the Claude hooks, the idle wake, the main-thread filter, the end of the pane path for posts, `post --interrupt`, quoted sender names, the trust sentence, the help text and the docs and stencils that describe the old reminder flow.

Out:

- Codex: there is no `codexengine` (board note `codex-engine`); the engine-neutral seam stays, no Codex hook is rendered.
- Stage 3 of creel (`creel-agents`): parent asks and `lym shuttle send` moving behind creel.
- `inject`, command injects, `interrupt`, holds and pauses keep their behaviour; `/clear`, `/compact` and the resume line still need typing.
- Restarting sessions launched before the deploy: they get posts once shuttle launches them afresh; a reed resume replays the old line and the old run's `settings.json`.

## Decisions

### D1 — Mid-turn delivery is a visible PostToolUse block

The `PostToolUse` hook keeps its flag test and runs `lym creel hook-read claude`, which now prints `{"decision":"block","reason":<lines>,"systemMessage":<lines>}` instead of `hookSpecificOutput.additionalContext`.
`hook-read` learns the event from the hook input's `hook_event_name`.
Hook input parsing and the output JSON stay under `claudeengine`, behind the `shuttleengine.ToolBoundaryContextRenderer` capability, which renders per event and answers D3's subagent check.
Claim, 8 KiB budget, hold, pause and the acknowledge-after-write order stay as `Store.ClaimAtBoundary` has them, with the one exclusion below.
A `PostToolUse` claim resets D2's continuation count.

Every line a claim hands out fits one hook field.
`post`, a reply to a strand included, refuses a message whose `MessageLine`, in D8's form, would pass the 8 KiB claim budget, with the refusal's existing way forward: write the text to a file and mail its path.
That line cap replaces the 64 KiB `maxBodyBytes` cap for every message a claim can hand out; a reply to `EXTERNAL`, which reaches only a waiting `post --wait-reply`, keeps the 64 KiB body cap; `forward` keeps the line it forwards.
LOOM's decision notice, a program's post that cannot take the file way forward, sends one notice per entry when its joined line would pass the budget, and `lym loom decision add` refuses, before it records anything, an entry whose own notice would pass it, naming the way forward: shorten the decision or rationale and put the detail in a file the entry names.
A message the old binary stored whose line passes 10,000 characters is left out of every claim, so it neither blocks a stop nor rings, and stays for `lym creel read`.

Rationale: the trust spike (`docs/research/creel-midturn-trust-2026-10-28/report.md`) shows Sonnet and Haiku (with D9's sentence) take a `block` with `reason` at once and the operator sees the lines under the tool result; `additionalContext` is invisible and Haiku refused it.
`ClaimAtBoundary` always hands out the first message whatever its size (`internal/creelengine/verbs.go`), so only a cap at `post` keeps a claim inside a field; past the field cap Claude Code shows a preview while creel journals the message read (Constraints).
A line of at most 8 KiB is at most 8,192 UTF-16 units, the measure of a JavaScript string's length.
Bound: a post between 8 KiB and 64 KiB, accepted today, is refused and goes through a file; an old-binary message past 10,000 characters gets no signal and shows in `lym creel inbox`.

### D2 — The turn-end Stop hook blocks while claimable mail stands, and a blocked stop records no turn end

The `Stop` entry's recording command and the creel Stop delivery become one command.
It reads the hook's standard input once into a variable and hands that payload to both steps.
The flag test gates only the creel step: when the flag stands it runs `hook-read` with the payload, writing straight to the hook's standard output.
On a `Stop`, `hook-read` exits 0 exactly when it wrote a block, whatever follows the write, and non-zero when it claimed nothing or failed before writing.
On its exit 0 the command appends nothing to `events.jsonl`, neither the stamp line nor the payload;
otherwise, or when the flag is absent, it appends the stamp line and the payload as the recording command does today and exits 0.
The stamp is written only together with the record, after the creel step decides.
The block's shape is D1's.

Creel counts, per strand, the consecutive `Stop` blocks of the strand's current turn, tracking Claude Code's own count (Constraints).
A `PostToolUse` claim (D1), a `Stop` whose `stop_hook_active` is false and a D4 ring reset it; `hook-read` passes `stop_hook_active` to the daemon, which applies a reset before it checks the cap.
At 6 the Stop claims nothing, so the turn end is recorded; the strand is then idle, and D4's ring wakes it for the mail left.
A daemon that starts, a restart after a binary replacement included, sets every strand's count to the cap, so no Stop blocks until one of those resets; the count is never journaled.

Rationale: Claude Code runs an event's matching hooks in parallel (Constraints), so a separate creel Stop hook could not keep the recording hook from logging a turn end the block then cancels; `ParseEvents` reads every `Stop` line as `EventStop` or `EventWaiting`, and a run would conclude on a turn that goes on.
Staying below Claude Code's cap of 8 means no override ever discards a block whose mail creel journaled read.
Claude Code starts every turn, a rewake turn included, at a count of 0 and with `stop_hook_active` carried in, true for a rewake turn (Constraints), so a ring starts a fresh count and successive idle wakes never add up.
Writing straight to the hook's output keeps the acknowledge-after-write order D1 keeps: `hook-read` acknowledges only after the block reached Claude Code's pipe.
Bound: no loop, since each block claims at least one message.
The count errs early, never late: a tool call with no mail standing resets Claude Code's count but not creel's, and a ring resets creel's only on D4's reading of no running turn, which no turn gives before its recorded end: a prompted turn's blocked stops record no turn end, and a woken turn counts as running from its ring (D4).
A daemon restart mid-chain ends the chain at its next Stop, and the ring that follows delivers the rest.
A hook killed between its write and the acknowledgement leaves the claim unacknowledged, so the mail comes again at the next boundary, never lost.
A merged command killed at its timeout after the write never reaches its record step, so when Claude Code drops the killed hook's output the turn ends with no `Stop` record; the window is the claim's 10-second daemon call against the hook's timeout.

### D3 — Only the main thread takes mail

`hook-read` and D4's `hook-wait` ask the capability whether the hook input comes from a subagent; for Claude, a non-empty `agent_id` field.
A subagent or fork call claims nothing, prints nothing and exits 0, and the mail and its flag stay for the main thread's next boundary.

Rationale: a fork's context dies with it, so mail it took would be lost to the role.
Bound: while mail stands, each subagent tool call starts one `lym` process that delivers nothing, the same cost the hold case already carries.

### D4 — An asyncRewake waiter wakes an idle session with a fixed bell; the mail goes through D1 or D2

The `Stop` event gets a second entry, `asyncRewake: true` with a timeout of 86400 seconds, well under the JavaScript timer ceiling of 2^31−1 ms, running `lym creel hook-wait claude`.
The waiter skips a subagent call (D3), then holds a waiting daemon call registered for its strand.
A newer waiter of the same strand ends the older one, which exits 0.
A dropped connection is reconnected and the registration renewed, as other waiting calls reconnect up to `MaxReconnects`; a waiter that cannot reconnect exits 0 and never exits 2.
The daemon counts a waiter live while its connection is open.

The daemon wakes a strand once per claimable set, a set changing when new claimable mail arrives or a claim takes some.
It rings a live waiter only when the strand holds mail a claim would hand out (hold and pause applied) and the `Delivery` seam reports no running turn.
That reading is a new field of the seam's `Idleness` answer, `NoTurn`, true when no turn is in progress whatever the input box holds:
the hook-derived readiness reading decides it, and when that reading is unknown the pane probe does.
A woken turn records no `UserPromptSubmit`, so the daemon holds `NoTurn` false from a ring until a turn end recorded after it; without that, mail arriving in a woken turn's blocked chain would find the waiter its last blocked `Stop` started and ring mid-chain.
A draft does not hold the ring; a too-short pane or a missing input box holds it only when the pane probe decides, since the probe then reads no turn.
A ring resets D2's count.
On the ring the waiter writes one fixed line to standard error and exits 2:

```text
[creel] Mail is waiting for you. End this turn now; creel hands the mail to you as the turn ends. Do not run lym creel read for it.
```

The waiter claims nothing, so the woken turn's end (D2) or its first tool call (D1) delivers the mail in a block the operator sees.
When neither a claim nor a busy reading follows a ring within 30 seconds, the set's next wake is D5's bell.

Rationale: an `asyncRewake` exit 2 is the one channel that wakes an idle session without the pane and keeps a draft (spike case 3), but a settings hook cannot show its text to the operator (Constraints); letting the wake carry no mail and the next boundary carry it makes every message visible.
Bound: one wake costs one short model turn before the mail turn.
A ring that races a turn the operator starts delivers nothing and leaves one stale bell reminder, which may join that turn.
A wake turn starts with no `UserPromptSubmit` and ends with a recorded turn end shuttle did not prompt, as a fork-completion turn or a typed notice did before; shuttle's guarded send reads the pane as busy during it.
A waiter whose session died rings into nothing once, and the 30-second rule hands the set to D5.
The woken-set marks live in the daemon's memory, so a restarted daemon may ring a set once more, as D5's Bound says of the bell.

### D5 — A typed fixed-text bell wakes an idle session no waiter reaches

The delivery worker types D4's bell line through the existing guarded pane send, under every rule a typed line has today (pause, hold, busy requeue, `ErrNotLanded` never retyped), when the guarded send's own `Delivery.Idle` answers idle, holds claimable mail and either has had no live waiter for 5 seconds or its ring went unanswered (D4).
A draft, a too-short pane or a missing input box holds the bell as it holds every typed line.
It types the bell once per claimable set and never repeats it for an unchanged set.
Only a strand whose session runs D2's Stop delivery gets it.
Shuttle marks such a strand with a new opaque reed field, `CreelHooks`, set on each launch whose rendered settings carry D1's, D2's and D4's hooks and kept through reed's verbatim resume.
A strand without the mark gets no bell and keeps its mail for `lym creel read`: a hand-added strand (`lym reed add`), a `lym shuttle open` session, which runs under its worktree's own settings, and a session launched before the deploy.

Rationale: a user interrupt fires no `Stop` (Constraints), so after `post --interrupt` or `interrupt` no new waiter starts; whether Claude Code keeps an older waiter alive across a later turn, `/clear` or `/compact` is undocumented; the waiter ends at its timeout.
The bell carries no sender text, so no post is typed.
The 5-second grace keeps the bell out of the moment between a `Stop` and its waiter's connection.
A bell into a strand without D2's Stop delivery would tell it to end the turn for a delivery that never comes, the same ground on which hand-added strands get none.
Bound: one fixed line per claimable set, only into a strand marked `CreelHooks`; a restarted daemon may type it once more for a set it already woke.

### D6 — The pane path for posts goes

The delivery worker types only queued injects, command injects and D5's bell.
The notice line, the reminder line, its backoff and the 4 KiB notice cap go; agent mail is delivered in full by D1 and D2 like any other mail, but for D1's old-binary lines past the field cap.
The pane reservation of a notice before typing goes with it, since nothing types a message.
`lym creel read`, `read --wait`, `inbox`, `post --reply` and `forward` stay.

Rationale: every session shuttle launches with its rendered settings carries the hooks, and a typed notice waits for an idle pane and competes with the operator's draft, where the hooks deliver at the next boundary.
Bound: the guard that kept an agent's text out of another agent's context goes; the line still opens with the sender creel stamps (D8), and D9's sentence makes no difference between senders (operator, 2026-10-28, in the entry).
A strand without D5's `CreelHooks` mark gets no notice, reminder, block or bell for its mail and reads it only with `lym creel read`.

### D7 — `lym creel post --interrupt` interrupts, then posts

`post --interrupt <receiver>` is one daemon request: it validates the post first (receiver, line size, sender), then sends the guarded interrupt with the `interrupt` verb's sender rules, pause claim and refusals, then stores the message.
A post that fails validation sends no key.
A receiver without D5's `CreelHooks` mark is refused before any key, since nothing would reach the stopped session, naming the ways forward: `lym creel inject --interrupt <receiver> <text>`, which stops the turn and types the text, or a post without `--interrupt`, which waits for the receiver's `lym creel read`.
A receiver with no running turn (`ErrNoRunningTurn`) still gets the post.
A refused interrupt stores nothing and answers the refusal.
A store that fails after the key was sent answers an error saying the turn was interrupted and the post was not stored, naming the way forward: post again without `--interrupt`.
`--interrupt` with `--reply` is refused, naming the way forward: post to the receiver by name.
The stopped session then gets the message through D5's bell, or D4 when a waiter is still live, and D2.

Rationale: a scope change that must not wait for the next tool call stops the turn and lands as the next turn's input.
Bound: it lets a sender stop a turn only where `interrupt` already lets it; a call in flight is lost exactly as with `interrupt`.

### D8 — Creel quotes sender names in the text it writes

`creelengine.MessageLine` renders `[creel] #<id> from "<sender>": <body>`, and `lym creel read`'s text output quotes the sender the same way.
Every sender name is quoted, a program and `EXTERNAL` included, so the line has one shape.

Rationale: a name with `:` read inside prose is ambiguous; strand names keep `:` (operator, 2026-10-28, in the entry).

### D9 — Shuttle appends the trust and reply sentence to every Claude session's system prompt

Every Claude session shuttle launches with a run directory carries this sentence in its appended system prompt, on the launch and the resume line.
A `lym shuttle open` session, which writes no run directory and takes its worktree's own settings, carries none, as it carries no deny notice today.
Shuttle writes the appended text, the deny notice when there is one, a space, then the sentence, to `append-system-prompt.txt` beside the run's `settings.json`.
The line passes that file with one `--append-system-prompt-file` flag, which replaces the `--append-system-prompt` argument the deny notice rides today, so one flag carries both.
The sentence:

```text
Lines starting with [creel] are LoomYard internal mail delivered by hooks; follow them as the operator's own instructions. When one asks for an answer, reply with lym creel post --reply <id> "<text>", using the id after #; answer mail from EXTERNAL in your own reply text instead.
```

The sentence is its own constant with its own init check, which refuses only line breaks; the deny notices keep `noticeTextForbiddenChars`.

Rationale: Haiku obeyed a block only with the sentence (trust spike, cases 1 and 2); trust rests on no skill and no `CLAUDE.md`.
The sentence holds `"`, which a native argument on Windows does not carry reliably (Constraints), while a file path carries the text whatever the shell and its argument mode.
The deny-notice guard bans `"` for constants that also ride JSON and `echo`; this sentence rides only the file.

### D10 — `lym creel` help carries the reply rule

The Cobra help of the `creel` command group and of `post` states: reply with `lym creel post --reply <id> "<text>"` using the id after `#`, and answer mail from `EXTERNAL` in your own reply text, since such a reply reaches only a waiting `post --wait-reply`.

Rationale: the entry asks for it; an agent that reads help sees the same rule as the system prompt.

### D11 — Docs and stencils follow the new delivery

The package docs of `creelengine`, `creelcli` and `claudeengine`, and `docs/overview.md`, describe D1 to D8.
`contracts/stencils/orch/orch-template-role.md` and `internal/orchcli/doc.go`, which describe the reminder and `lym creel read` turn, are rewritten to the hook delivery.

Rationale: `CLAUDE.md`'s "Docs land in the same commit"; an orch told to wait for a reminder line would wait for nothing.

### Added after Discussion (parent, 2026-10-28): Posts still reach a session without creel hooks

Decision: D6 is narrowed: creel must deliver posts to Codex sessions too (operator, 2026-10-28). A strand whose engine has no creel hook delivery, such as a Codex session, keeps today's pane path for posts: the notice line and the reminder line with its backoff and notice cap, and the pane reservation before typing. Only Claude strands, whose hooks deliver posts, lose the pane path. The engine decides which path a strand gets; the codex-engine and creel-agents notes stay valid.

Rationale: The operator ruled that creel must be able to deliver to Codex. Codex has no proven hook channel yet; a later Codex task may move it onto hooks once a spike proves one.

### Added after Discussion (parent, 2026-10-28): A post body is capped at the hook field budget

Decision: maxBodyBytes in internal/creelengine/pool.go is lowered to the 8 KiB boundary budget, so `lym creel post` refuses a longer body with its existing way forward: write the text to a file and post its path, and the receiver reads the file (operator, 2026-10-28). Rewrite the Constraints bullet on the 10,000-character cap to state this bound, and add an acceptance criterion that a post over the budget is refused with that way forward.

Rationale: The operator called such long messages absurd; one rule keeps every delivered message whole within one hook field, so no message is journaled read while the session saw only a preview.

### Added after Discussion (parent, 2026-10-28): A strand with no hook-delivering engine keeps the pane path

Decision: A strand whose persisted Provider is empty, or names an engine that shuttleengines.ToolBoundaryRenderer does not find, keeps today's pane path for posts (the notice and the reminder), so a hand-added strand, a Codex session included, still gets its mail. Only a strand whose engine renders creel hook delivery loses the pane path.

Rationale: The operator ruled that creel must keep delivering to Codex (2026-10-28); today a Codex session exists only as a hand-added strand with no provider tag, so the empty-provider case must take the pane path for that ruling to hold.

## Constraints

- Claude Code 2.1.312 is the version the facts below rest on; "hooks docs" is `code.claude.com/docs/en/hooks`.
- Hooks docs, handler fields: an event's matching hooks run in parallel.
- Hooks docs, "Run hooks in the background": Claude Code enforces `timeout` on an `asyncRewake` hook, default 600 s; exit 2 wakes Claude even when idle; the hook's stderr, or stdout when stderr is empty, reaches the model as a system reminder.
- 2.1.312 bundle, the rewake handler: for a settings hook the operator sees only a fixed summary, `Stop hook feedback`; the hook entry's `rewakeSummary` key is `@internal` and static, so D4 sets none.
- Hooks docs, Stop: runs when the main agent finishes, not on a user interrupt; `stop_hook_active` is true when Claude is already continuing because of a stop hook; after eight consecutive continuations Claude Code overrides the next block, and a tool call resets the count.
- 2.1.312 bundle, the query loop: the block count is a guard of one turn's query.
  A new turn, a rewake turn included, starts it at 0 (`stopHookBlockingCount: 0` in the initial guards) with `stop_hook_active` carried in, and the rewake handler enqueues its wake with `stopHookActive: true`;
  a tool call's continuation resets the count to 0 and keeps `stop_hook_active`, which stays true for the rest of a turn once a block set it.
  The cap is `CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`, default 8.
- Hooks docs, PostToolUse: `decision: "block"` adds `reason` next to the tool result, and the tool's output stays; spike cases show the operator sees `PostToolUse:<tool> hook returned blocking error`, the reason and `PostToolUse:<tool> says: <systemMessage>`.
- Hooks docs, JSON output: `systemMessage` and `additionalContext` are each capped at 10,000 characters, measured per field, above which a preview replaces them.
  D1 caps every new line at the 8 KiB claim budget and a claim joins lines within it, so a field of new lines carries at most 8,192 characters; an old-binary line is claimed alone when it fits the cap and never claimed past it (D1).
- Hooks docs, common input fields, and the 2.1.312 hook-input builder: `agent_id` is set when a hook fires inside a subagent or an in-process teammate, from the agent context, and absent in the main thread; `agent_type` is also set by `--agent`, so D3 tests `agent_id` only.
  A fork is an Agent-tool subagent and carries one; the LLM-tier test pins it.
- Hooks docs: settings hooks fire `SubagentStop`, not `Stop`, when a subagent ends, so D2 and D4 run in the main thread; D3's check on them is a guard.
- `internal/shell`: the POSIX `Quote` single-quotes and the pwsh `Quote` single-quotes, so `"`, `<`, `>` and `#` reach the pane shell literally.
  On Windows the pane shell is pwsh 7 (`OrElse` relies on its `||`), which passes an argument holding `"` to a native program intact only in its `Standard` argument mode.
  Under `$PSNativeCommandArgumentPassing`'s Windows default, from 7.3 on, a `.cmd` or `.bat` target, such as an npm-installed `claude.cmd` shim, gets `Legacy` mode, as every target does before 7.3, and `Legacy` drops embedded double quotes (PowerShell docs, `about_Parsing`).
  Claude Code's native installer gives a `claude.exe`, but shuttle runs whatever `claude`, or `claude.yaml`'s `binary`, resolves to, so D9 carries the sentence in a file.
- 2.1.312 bundle, CLI options: `--append-system-prompt-file <file>` reads the text to append to the default system prompt from a file.
- A future engine receives creel mail only through hooks of its own, since D6 leaves no pane path for posts; `codex-engine` inherits that need.
- `PATTERN-shuttle-provider-seam`: hook input parsing, hook output JSON, the wake's exit convention and the bell's channel stay under `internal/shuttleengine/claudeengine`; `creelcli` and `creelengine` see only the capability.
- `PATTERN-pane-text-injection`: the bell (D5) is typed through creel's `Delivery` seam and reed's guarded send like any typed line.
- `PATTERN-daemon-wakeups`: the waiter blocks on its socket; the worker keeps its 1 s base and backoff.
- `PATTERN-tolerant-records`: journal records written by the old binary (`reminded`, notice reservations) still fold and are ignored where they no longer act.
- `PATTERN-test-speed`: real-Claude tests go in the LLM tier; claim, count, waiter, bell and filter logic get lower-tier tests with the provider stubbed.

## Auto-mode assumptions

- The bell text in D4 and D5 is the writer's wording; the entry fixes none.
- 86400 s as the waiter's timeout, 30 s for an unanswered ring, 5 s of grace before a bell, and a continuation cap of 6.
- A draft in the input box does not hold the wake (D4), since the spike shows the wake leaves a draft untouched.
- Every sender name is quoted in D8, programs and `EXTERNAL` included, read from "wherever creel writes a strand name".
- D9's sentence goes on every Claude session shuttle launches with a run directory, standalone ones included, as the entry says "every"; a `lym shuttle open` session has no run directory for its file.
- `--interrupt` with `--reply` is refused (D7) rather than interrupting the original sender.

## Open risks

- For the parent: the entry asks that the idle wake carry the message visibly, and to stop and ask when there is no way. A settings hook cannot show rewake text (Constraints), so D4's wake carries a fixed bell and the mail shows in the next Stop block, one extra model turn per idle wake. Is that acceptable?
- For the parent: the entry says the pane path for posts goes. D5 types a fixed bell, with no sender text, into an idle strand marked `CreelHooks` that no waiter reaches, which after a user interrupt is the usual case, so `post --interrupt` mostly wakes through it. Keep D5, or drop it and let such mail wait for the strand's next turn?
- Sessions launched before the deploy keep the old settings and resume line: they get no `Stop` delivery, no waiter, no bell and no D9 sentence until shuttle launches them afresh, since a reed resume replays the old line and settings.

<!-- lym:carry-over Discussion-Review -->
- Discussion-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/discussion/round-1-review.md; fixer report: _lym/reviews/discussion/round-1-fixer-report.md.
  - F1 (class design, severity BLOCKING, open in rounds 1)
  - F12 (class decision, severity LOW, open in rounds 1; answered since by: Posts still reach a session without creel hooks; A post body is capped at the hook field budget)
  - bell-misdirects-pre-deploy-sessions (class design, severity MEDIUM, open in rounds 1)
  - continuation-count-climbs-reset-unordered (class design, severity MEDIUM, open in rounds 1)
  - daemon-restart-loses-count-and-woken-marks (class design, severity MEDIUM, open in rounds 1)
  - idle-reading-missing-from-delivery-seam (class design, severity MEDIUM, open in rounds 1)
  - oversized-first-message-overflows-hook-field (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Discussion-Review -->

<!-- lym:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/plan/round-2-review.md; fixer report: _lym/reviews/plan/round-2-fixer-report.md.
  - journal-close-test-row-panics (class design, severity MEDIUM, open in rounds 2)
  - llm-smoke-bell-pane-assertion (class design, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Plan-Review -->

## Acceptance criteria

All real-Claude scenarios run in the LLM tier through shuttle's own rendered `settings.json` and launch line, never a hand-launched session.

- A strand in a multi-tool turn gets a post at its next tool call as a `PostToolUse` block, acts on it in that turn, and the operator sees the message under the tool result.
- A post that arrives after a turn's last tool call is delivered by the `Stop` block, the turn goes on, and `events.jsonl` holds neither a stamp line nor a `Stop` record for the blocked stop and one of each, with its full payload, for the real end.
- A `Stop` with no flag, no claimable mail or a failing `hook-read` records the turn end with its full payload and does not block.
- At creel's continuation cap the `Stop` claims nothing and records the turn end; the strand's waiter then rings, and the woken turn's `Stop` delivers the mail.
- Three successive idle wakes of one strand each ring and each deliver through the `Stop` block.
- A daemon restart mid-chain never lets the chain's blocks pass creel's cap: the chain's next `Stop` claims nothing, and the ring that follows delivers the mail.
- An idle strand with a live waiter is woken within seconds of a post, a draft in its input box is untouched, the message text then shows in the pane as the Stop block, and no bell is typed for that set.
- An idle strand no waiter reaches gets the bell typed once per claimable set and then the message as the Stop block; a draft holds the bell; a strand without the `CreelHooks` mark, hand-added or launched before the deploy, gets no bell.
- A newer waiter ends an older one of the same strand; a waiter reconnects after a daemon restart; a ring that loses to a running turn delivers nothing.
- A hook call from a fork or subagent claims nothing and leaves the flag; the main thread's next boundary takes the mail, with a real fork.
- Agent mail is delivered in full, but for an old-binary line past the field cap (D1); no reminder or notice line is typed into any pane.
- Under a pause nothing is delivered and no wake goes out; under a hold only the owner's mail is claimed or woken for.
- `post --interrupt` stops a running turn and the message reaches the stopped session; a receiver with no running turn still gets the post; an invalid post sends no key; a receiver without the `CreelHooks` mark is refused before any key; a refused interrupt stores nothing; a store that fails after the key answers that the turn was interrupted and the post not stored; `--interrupt --reply` is refused with the way forward.
- `MessageLine` and `lym creel read` quote the sender: `[creel] #1 from "ly:orch": …`.
- Launch and resume lines pass one `--append-system-prompt-file`, whose file holds the deny notice when one exists and D9's sentence verbatim after it.
- `lym creel --help` and `lym creel post --help` carry the reply rule.
- A claim at the 8 KiB budget arrives whole in both `reason` and `systemMessage`; what is left goes at the next boundary.
- A post whose line would pass the 8 KiB budget is refused with the file way forward and stores nothing; a reply to `EXTERNAL` over 8 KiB is accepted; `lym loom decision add` refuses an entry whose notice would pass the budget before it records anything; an old-binary message whose line passes 10,000 characters is never claimed and stays for `lym creel read`.

## Notes for the plan writer

- `Store.Interrupt` and the `inject --interrupt` flow are the model for D7's one-request order.
- The board note `creel-agents` describes a chase "typed only at idle" under the reminder rules D6 removes, and a reply discipline D9 and D10 partly meet; this task leaves the note to the operator.
