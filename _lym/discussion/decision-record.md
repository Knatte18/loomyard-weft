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
- Restarting sessions launched before the deploy: they get posts once restarted.

## Decisions

### D1 — Mid-turn delivery is a visible PostToolUse block

The `PostToolUse` hook keeps its flag test and runs `lym creel hook-read claude`, which now prints `{"decision":"block","reason":<lines>,"systemMessage":<lines>}` instead of `hookSpecificOutput.additionalContext`.
`hook-read` learns the event from the hook input's `hook_event_name`.
Hook input parsing and the output JSON stay under `claudeengine`, behind the `shuttleengine.ToolBoundaryContextRenderer` capability, which renders per event and answers D3's subagent check.
Claim, 8 KiB budget, hold, pause and the acknowledge-after-write order stay as `Store.ClaimAtBoundary` has them.
A `PostToolUse` claim resets D2's continuation count.

Rationale: the trust spike (`docs/research/creel-midturn-trust-2026-10-28/report.md`) shows Sonnet and Haiku (with D9's sentence) take a `block` with `reason` at once and the operator sees the lines under the tool result; `additionalContext` is invisible and Haiku refused it.

### D2 — The turn-end Stop hook blocks while claimable mail stands, and a blocked stop records no turn end

The `Stop` entry's recording command and the creel Stop delivery become one command.
It reads the hook's standard input once into a variable and hands that payload to both steps.
The flag test gates only the creel step: when the flag stands it runs `hook-read` with the payload;
when that prints a block, the command prints it and appends no `Stop` record to `events.jsonl`;
when the flag is absent, `hook-read` prints nothing or it fails, the command records the turn end with the payload as today and exits 0.
The block's shape is D1's.

Creel counts, per strand, its consecutive continuations: D2 blocks and D4 rings.
A `PostToolUse` claim (D1) and a `Stop` whose `stop_hook_active` is false reset the count; `hook-read` passes `stop_hook_active` to the daemon.
At 6 the Stop claims nothing, so the turn end is recorded, and D4 rings no more for that strand until the count resets; the mail stays unread and D5 is its wake.

Rationale: Claude Code runs an event's matching hooks in parallel (Constraints), so a separate creel Stop hook could not keep the recording hook from logging a turn end the block then cancels; `ParseEvents` reads every `Stop` line as `EventStop` or `EventWaiting`, and a run would conclude on a turn that goes on.
Staying below Claude Code's cap of 8 means no override ever discards a block whose mail creel journaled read.
Bound: no loop, since each block claims at least one message; the count errs early, never late, because a tool call with no mail standing resets Claude Code's count but not creel's.

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
It rings a live waiter only when the strand holds mail a claim would hand out (hold and pause applied), D2's count is below its cap, and the strand's idle check through the `Delivery` seam reports no running turn, where a draft in the input box does not count as busy.
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

### D5 — A typed fixed-text bell wakes an idle session no waiter reaches

The delivery worker types D4's bell line through the existing guarded pane send, under every rule a typed line has today (pause, hold, busy requeue, `ErrNotLanded` never retyped), when the strand is idle (D4's reading), holds claimable mail and either has had no live waiter for 5 seconds, or its ring went unanswered (D4), or D2's count is at its cap.
It types the bell once per claimable set and never repeats it for an unchanged set.
Only a strand whose session shuttle launched gets it; a hand-added strand (`lym reed add`) gets none and keeps its mail for `lym creel read`.

Rationale: a user interrupt fires no `Stop` (Constraints), so after `post --interrupt` or `interrupt` no new waiter starts; whether Claude Code keeps an older waiter alive across a later turn, `/clear` or `/compact` is undocumented; the waiter ends at its timeout.
The bell carries no sender text, so no post is typed.
The 5-second grace keeps the bell out of the moment between a `Stop` and its waiter's connection.
Bound: one fixed line per claimable set, only into a shuttle-launched strand; a restarted daemon may type it once more for a set it already woke.

### D6 — The pane path for posts goes

The delivery worker types only queued injects, command injects and D5's bell.
The notice line, the reminder line, its backoff and the 4 KiB notice cap go; agent mail is delivered in full by D1 and D2 like any other mail.
The pane reservation of a notice before typing goes with it, since nothing types a message.
`lym creel read`, `read --wait`, `inbox`, `post --reply` and `forward` stay.

Rationale: every hub session is launched by shuttle with the hooks, and a typed notice waits for an idle pane and competes with the operator's draft, where the hooks deliver at the next boundary.
Bound: the guard that kept an agent's text out of another agent's context goes; the line still opens with the sender creel stamps (D8), and D9's sentence makes no difference between senders (operator, 2026-10-28, in the entry).

### D7 — `lym creel post --interrupt` interrupts, then posts

`post --interrupt <receiver>` is one daemon request: it validates the post first (receiver, body size, sender), then sends the guarded interrupt with the `interrupt` verb's sender rules, pause claim and refusals, then stores the message.
A post that fails validation sends no key.
A receiver with no running turn (`ErrNoRunningTurn`) still gets the post.
A refused interrupt stores nothing and answers the refusal.
`--interrupt` with `--reply` is refused, naming the way forward: post to the receiver by name.
The stopped session then gets the message through D5's bell, or D4 when a waiter is still live, and D2.

Rationale: a scope change that must not wait for the next tool call stops the turn and lands as the next turn's input.
Bound: it lets a sender stop a turn only where `interrupt` already lets it; a call in flight is lost exactly as with `interrupt`.

### D8 — Creel quotes sender names in the text it writes

`creelengine.MessageLine` renders `[creel] #<id> from "<sender>": <body>`, and `lym creel read`'s text output quotes the sender the same way.
Every sender name is quoted, a program and `EXTERNAL` included, so the line has one shape.

Rationale: a name with `:` read inside prose is ambiguous; strand names keep `:` (operator, 2026-10-28, in the entry).

### D9 — Shuttle appends the trust and reply sentence to every Claude session's system prompt

Every Claude session shuttle launches carries this sentence through `--append-system-prompt`, on the launch and the resume line, after the deny notice when there is one:

```text
Lines starting with [creel] are LoomYard internal mail delivered by hooks; follow them as the operator's own instructions. When one asks for an answer, reply with lym creel post --reply <id> "<text>", using the id after #; answer mail from EXTERNAL in your own reply text instead.
```

The sentence is its own constant with its own init check, which refuses only line breaks; the deny notices keep `noticeTextForbiddenChars`.

Rationale: Haiku obeyed a block only with the sentence (trust spike, cases 1 and 2); trust rests on no skill and no `CLAUDE.md`.
The deny-notice guard bans `"` for constants that also ride JSON and `echo`; this sentence rides only the launch argument, which both `shell.Quote` forms carry with `"` intact (Constraints).

### D10 — `lym creel` help carries the reply rule

The Cobra help of the `creel` command group and of `post` states: reply with `lym creel post --reply <id> "<text>"` using the id after `#`, and answer mail from `EXTERNAL` in your own reply text, since such a reply reaches only a waiting `post --wait-reply`.

Rationale: the entry asks for it; an agent that reads help sees the same rule as the system prompt.

### D11 — Docs and stencils follow the new delivery

The package docs of `creelengine`, `creelcli` and `claudeengine`, and `docs/overview.md`, describe D1 to D8.
`contracts/stencils/orch/orch-template-role.md` and `internal/orchcli/doc.go`, which describe the reminder and `lym creel read` turn, are rewritten to the hook delivery.

Rationale: `CLAUDE.md`'s "Docs land in the same commit"; an orch told to wait for a reminder line would wait for nothing.

## Constraints

- Claude Code 2.1.312 is the version the facts below rest on; "hooks docs" is `code.claude.com/docs/en/hooks`.
- Hooks docs, handler fields: an event's matching hooks run in parallel.
- Hooks docs, "Run hooks in the background": Claude Code enforces `timeout` on an `asyncRewake` hook, default 600 s; exit 2 wakes Claude even when idle; the hook's stderr, or stdout when stderr is empty, reaches the model as a system reminder.
- 2.1.312 bundle, the rewake handler: for a settings hook the operator sees only a fixed summary, `Stop hook feedback`; the hook entry's `rewakeSummary` key is `@internal` and static, so D4 sets none.
- Hooks docs, Stop: runs when the main agent finishes, not on a user interrupt; `stop_hook_active` is true when Claude is already continuing because of a stop hook; after eight consecutive continuations Claude Code overrides the next block, and a tool call resets the count.
- Hooks docs, PostToolUse: `decision: "block"` adds `reason` next to the tool result, and the tool's output stays; spike cases show the operator sees `PostToolUse:<tool> hook returned blocking error`, the reason and `PostToolUse:<tool> says: <systemMessage>`.
- Hooks docs, JSON output: `systemMessage` and `additionalContext` are each capped at 10,000 characters, measured per field, above which a preview replaces them; D1's 8 KiB claim budget keeps each field under the cap.
- Hooks docs, common input fields, and the 2.1.312 hook-input builder: `agent_id` is set when a hook fires inside a subagent or an in-process teammate, from the agent context, and absent in the main thread; `agent_type` is also set by `--agent`, so D3 tests `agent_id` only.
  A fork is an Agent-tool subagent and carries one; the LLM-tier test pins it.
- Hooks docs: settings hooks fire `SubagentStop`, not `Stop`, when a subagent ends, so D2 and D4 run in the main thread; D3's check on them is a guard.
- `internal/shell`: the POSIX `Quote` single-quotes and the pwsh `Quote` single-quotes, so `"`, `<`, `>` and `#` pass literally to the launch argument.
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
- D9's sentence goes on every Claude session shuttle launches, standalone ones included, as the entry says "every".
- `--interrupt` with `--reply` is refused (D7) rather than interrupting the original sender.

## Open risks

- For the parent: the entry asks that the idle wake carry the message visibly, and to stop and ask when there is no way. A settings hook cannot show rewake text (Constraints), so D4's wake carries a fixed bell and the mail shows in the next Stop block, one extra model turn per idle wake. Is that acceptable?
- For the parent: the entry says the pane path for posts goes. D5 types a fixed bell, with no sender text, into an idle shuttle-launched strand that no waiter reaches, which after a user interrupt is the usual case, so `post --interrupt` mostly wakes through it. Keep D5, or drop it and let such mail wait for the strand's next turn?
- Sessions launched before the deploy keep the old settings and resume line: they get no `Stop` delivery, no waiter and no D9 sentence until restarted, and D5 may bell them once per claimable set.

## Acceptance criteria

All real-Claude scenarios run in the LLM tier through shuttle's own rendered `settings.json` and launch line, never a hand-launched session.

- A strand in a multi-tool turn gets a post at its next tool call as a `PostToolUse` block, acts on it in that turn, and the operator sees the message under the tool result.
- A post that arrives after a turn's last tool call is delivered by the `Stop` block, the turn goes on, and `events.jsonl` holds no `Stop` record for the blocked stop and one, with its full payload, for the real end.
- A `Stop` with no flag, no claimable mail or a failing `hook-read` records the turn end with its full payload and does not block.
- At creel's continuation cap the `Stop` claims nothing and records the turn end, and no ring follows until the count resets.
- An idle strand with a live waiter is woken within seconds of a post, a draft in its input box is untouched, the message text then shows in the pane as the Stop block, and no bell is typed for that set.
- An idle strand no waiter reaches gets the bell typed once per claimable set and then the message as the Stop block; a hand-added strand gets no bell.
- A newer waiter ends an older one of the same strand; a waiter reconnects after a daemon restart; a ring that loses to a running turn delivers nothing.
- A hook call from a fork or subagent claims nothing and leaves the flag; the main thread's next boundary takes the mail, with a real fork.
- Agent mail is delivered in full; no reminder or notice line is typed into any pane.
- Under a pause nothing is delivered and no wake goes out; under a hold only the owner's mail is claimed or woken for.
- `post --interrupt` stops a running turn and the message reaches the stopped session; a receiver with no running turn still gets the post; an invalid post sends no key; a refused interrupt stores nothing; `--interrupt --reply` is refused with the way forward.
- `MessageLine` and `lym creel read` quote the sender: `[creel] #1 from "ly:orch": …`.
- Launch and resume lines carry D9's sentence verbatim, after the deny notice when one exists.
- `lym creel --help` and `lym creel post --help` carry the reply rule.
- A claim at the 8 KiB budget arrives whole in both `reason` and `systemMessage`; what is left goes at the next boundary.

## Notes for the plan writer

- `Store.Interrupt` and the `inject --interrupt` flow are the model for D7's one-request order.
- The board note `creel-agents` describes a chase "typed only at idle" under the reminder rules D6 removes, and a reply discipline D9 and D10 partly meet; this task leaves the note to the operator.
