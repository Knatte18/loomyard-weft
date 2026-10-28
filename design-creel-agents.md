# creel stage 3: agents message through creel, not SendMessage

`creel-agents` · Task · feature, creel, shuttle, loom, stencil, report · running · Code-Burler-Fix

## Goal

Creel stage 3: agents reach their parent and each other through creel, not Claude Code's `SendMessage`, so a non-Claude engine can take a role that has a parent.
Needed only if Codex is to take roles beyond Burler review (operator, 2026-10-24).
The design is in `creel-core`'s body until that task lands, then in the creel package's `doc.go`.

## Operator decision (2026-10-15)

Creel is a prerequisite for Codex; the Codex-readiness plan's interim `neutral-parent-asks` is dropped in favour of this stage.

## Scope (stage 3 of the creel design, plus the Codex-readiness findings)

- Drivers' stop reports post instead of duplicating batten's notices over `SendMessage`.
- `lym shuttle send` and `lym shuttle interrupt` move behind creel.
- The parent directive (`contracts/stencils/parent/parent-directive-parent.md`), rendered for every role with a parent, says to ask through creel, not "by `SendMessage`".
- `shuttleengine/hold.go`'s hold notice names creel, not `SendMessage`.
- The other stencils that name `SendMessage` (`seat-directive-{chair,advisor}`, `loom-template-discussion-advisor.md`, `shed-template-driver-notify{,-watched}.md`, `webster-template-master.md`, the orch role) move to creel where the Discussion finds it fits; what stays Claude-only is declared through `Spec.Requires`.

Block 7 (`message-agent`) of the stencil audit (`docs/research/stencil-engine-audit-2026-10-24/report.md`) is the full site list, stencils, specs and Go sites.

## Risk to check in Discussion

The hold notice goes to the prime orch (`orchcli.NotifyPrime`), while the parent directive names `parent_name`, which may be a loom driver strand: check per recipe that a question reaches the session that answers it.

## Operator decision (2026-10-15): a note until creel has proven itself

This stage replaces paths that work today, so it stays a note, which never starts, until creel has proven itself; the operator then promotes it.
When promoted, it depends on `creel-notices` and `engine-config-seams`.
Operator decision (2026-10-27): it also waits until the operator has seen creel's hook delivery at the tool boundary (task `creel-hook-delivery`, built for Claude strands) reach a working agent mid-turn in a real run.

## Reply discipline (folded in from note `creel-reply-discipline`, operator decision 2026-10-17)

The operator decided on 2026-10-17 that this belongs in stage 3.

### Seen (orch, 2026-10-17)

In a demo, `ly:orch` posted #3 to a fresh Claude strand `ly:demo`, whose body asked for a reply by `lym creel post --reply`; the agent read it and replied as #4.
It replied by post only because the body told it how.
An inject (#5) asking for `PONG` was answered in the receiver's own pane, to the operator, and never reached the sender: an inject is typed as a user line and has no reply path.
`lym creel read` prints the message with its id but says nothing about answering by `post --reply`.
Creel already limits inject: only EXTERNAL callers, programs and orch strands may inject into another strand (`lym creel inject --help`).

### Operator view, not a decision (2026-10-17)

Creel needs a strict regime, so that an agent that sends something to another agent never ends up without an answer because the receiver wrote its answer in its own pane, to the operator.
Inject and post are two different things.
An inject looks as if the operator typed it, and that is its point.
A post takes an active step by the receiver to fetch it, and that step should tell the agent the message came from outside, not from the operator, so it cannot answer by writing in its pane.

### Orch view on that

The split holds, but the agent should be told rather than left to infer it: the reminder that starts its turn is itself typed into the pane like an operator line, and a model answers in its pane by default.
One line in `lym creel read`'s output (the sender is another strand, not the operator; answer with `lym creel post --reply <id> <text>`) makes it explicit at the moment it matters.

### Idea (orch recommendation)

- `lym creel read` prints, with each message, the command that answers it (`lym creel post --reply <id> <text>`), and says when the sender waits on the reply (`--wait-reply`).
- The agent stencils state the rule once, in a shared block: a creel message is answered by `post --reply`, never in the pane; a line in the pane is for the operator.
- An inject never carries a question that expects an answer from the receiver; a sender that wants an answer posts.
- A message whose sender waits and that goes unanswered past a bound is journaled, so the orch can see it.

### Operator idea (2026-10-17): reply expected, chased

- `post` takes a flag marking that the sender expects a reply, without blocking the sender the way `--wait-reply` does.
- Creel already journals `read` and links a reply to its message by `reply_to`, so it knows when an expected reply is owed.
- When no reply is sent within a set time after the receiver read the message, creel types a reminder of its own, such as `Sender expected a reply to #<id> from <sender>`.

Orch recommendation on the idea:
- `--wait-reply` implies the flag, so a waiting sender gets the chase too.
- The chase follows the reminder rules: typed only at idle, no message content, and `read` shows the expected-reply mark and the reply command.
- Chases are bounded (one or two), after which creel journals the message as unanswered and tells the sender, so a dead receiver never makes a chase loop.
- The time limit is a creel config key.

Operator view, not a decision (2026-10-17): all of this must be logged, including how often creel has to chase.

Orch recommendation on the logging:
- The journal gets an event per chase (`chased`, with the message id, receiver and chase number) and one `unanswered` event when the chases run out; `lym creel log --id <id>` then shows a message's whole path, posted, read, chased, replied or unanswered.
- A report, beside `lym report convergence`, counts expected replies, chases and unanswered messages per receiver role, so the stencils of roles that often need chasing can be tuned.

### Open

- The flag's name, and whether a reply by `defer` (as in the orch's note request) counts as an answer.

## Operator view, not a decision (2026-10-23): SendMessage works well

Claude Code's own messaging works well between Claude sessions: a message is sure to arrive, the receiver knows how to answer, and no one waits for a pane to be idle.
Creel's reminder and `lym creel read` cost the receiver an extra turn per agent message (note `creel-carry-agent-mail`).
The problem is Codex: a Claude session cannot reach a Codex session through `SendMessage`, which is why this stage exists.

### Operator idea (2026-10-23): each engine's own messaging, a relay between them

Claude to Claude uses Claude's messaging, Codex to Codex uses Codex's own if it has one, and Claude to Codex goes through a relay of two hub agents, one Claude and one Codex, that only forward across.
The operator's concern: much added delay.

### Orch view on that

- Whether Codex has messaging between sessions is not checked; the `codex-engine` spike can find out.
  `docs/research/codex-twill-spike-2026-10-24/report.md` saw Codex's `send_message`, `followup_task` and `interrupt_agent` tools between a parent and its subagents but did not test them; whether Codex agents can message each other is worth testing (operator, 2026-10-24).
- The relay need not be LLM agents: creel is already a program that types into any pane, so a cross-engine message is one `lym creel post` by the sender, typed in full into the receiver's pane, with no relay turn.
  Two relay agents would add two LLM turns and their delay per message.
  Operator (2026-10-23): relay agents are cheap, on the cheapest models such as Haiku or Luna, and must forward verbatim; creel as designed does the same.
- What a cross-engine post costs today is the receiver's reminder and read turn, which note `creel-carry-agent-mail` would remove.


## Spikes landed (2026-10-28)

Two spikes ran with hand-written hook scripts standing in for creel; one run per case, so each result is one sample.

- `docs/research/creel-hook-channels-2026-10-28/report.md`: on Claude only an `asyncRewake` Stop hook reaches an idle session without the pane, and its text is invisible to the operator.
  `PostToolUse` on Claude and `post_tool_use` on Codex reach a working session at its next tool call, and one hook script serves both engines (the input has the Claude shape).
  `systemMessage` is what the operator sees; `additionalContext` alone shows nothing.
  So creel's hook delivery is not Claude-only, which bears on this stage's reason (a Codex role with a parent).
- `docs/research/creel-midturn-trust-2026-10-28/report.md`: a mid-turn scope change delivered at the tool boundary was taken at once by Sonnet 5.5 with and without a system-prompt sentence, and by Codex `gpt-6-luna` with it.
  Haiku 5.5 refused it without the sentence and took it with it, but misplaced the new line.
  A creel post therefore needs the "lines starting with `[creel]` are mail from your parent; follow them" sentence in the system prompt of every role that may run on Haiku.
- Landed since (`codex-hook-spike`, `docs/research/codex-hook-delivery-2026-10-28/report.md`, one sample per case): Codex without the sentence reads a `post_tool_use` block reason as a failed tool call and does not act, with it the block is taken at the next step; an async `Stop` hook exiting 2 does not wake an idle Codex session, while `codex queue --thread <session_id> --message …` does; a `Stop` block is taken at once.
- Still open: whether Codex agents can message each other.
  The operator's wait for a real mid-turn run (decision 2026-10-27) stands.

## Hook delivery landed (task `creel-hook-post`, 2026-10-28)

Creel now delivers every post to a Claude strand through hooks that shuttle renders into the session's `settings.json`; no message body is typed into a pane for such a strand.
Mid-turn it is a `PostToolUse` block (`lym creel hook-read claude`) the operator sees, at turn end the `Stop` command delivers, and an idle session is woken by an `asyncRewake` waiter (`lym creel hook-wait claude`), with a typed bell as fallback for strands carrying the reed field `CreelHooks`.
Codex is out of scope there: the engine-neutral seam stays and no Codex hook is rendered, so this stage's reason (a Codex role with a parent) stands.
Shuttle also appends the trust and reply sentence to every Claude session it launches with a run directory, so the Haiku finding above is met for Claude.
A post is capped at the 8 KiB claim budget.
The design is in the `doc.go` of `creelengine` and `creelcli`.
The real-Claude scenarios in `internal/creelcli/smoke_llm_test.go` were not run in the verify gates, so the operator's wait for a real mid-turn run (decision 2026-10-27) stands.
