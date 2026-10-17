# creel stage 3: agents message through creel, not SendMessage

`creel-agents` · Note · feature, creel, shuttle, loom, stencil, report

## Goal

Creel stage 3: agents reach their parent and each other through creel, not Claude Code's `SendMessage`, so a non-Claude engine can take a role that has a parent.
The design is in `creel-core`'s body until that task lands, then in the creel package's `doc.go`.

## Operator decision (2026-10-15)

Creel is a prerequisite for Codex; the Codex-readiness plan's interim `neutral-parent-asks` is dropped in favour of this stage.

## Scope (stage 3 of the creel design, plus the Codex-readiness findings)

- The parent-review gate (`internal/parentreview/gate.go`, which today prompts Discussion-Write to `SendMessage` the parent) posts itself.
- Drivers' stop reports post instead of duplicating batten's notices over `SendMessage`.
- `lyx shuttle send` and `lyx shuttle interrupt` move behind creel.
- The parent directive (`contracts/stencils/parent/parent-directive-parent.md`), rendered for every role with a parent, says to ask through creel, not "by `SendMessage`".
- `shuttleengine/hold.go`'s hold notice names creel, not `SendMessage`.
- The other stencils that name `SendMessage` (`seat-directive-{chair,advisor}`, `shed-template-driver-notify{,-watched}.md`, `loom-template-parent-review-delivery.md`, the orch role) move to creel where the Discussion finds it fits; what stays Claude-only is declared through `Spec.Requires`.

## Risk to check in Discussion

The hold notice goes to the prime orch (`orchcli.NotifyPrime`), while the parent directive names `parent_name`, which may be a loom driver strand: check per recipe that a question reaches the session that answers it.

## Operator decision (2026-10-15): a note until creel has proven itself

This stage replaces paths that work today, so it stays a note, which never starts, until creel has proven itself; the operator then promotes it.
When promoted, it depends on `creel-notices` and `engine-config-seams`.

## Reply discipline (folded in from note `creel-reply-discipline`, operator decision 2026-10-17)

The operator decided on 2026-10-17 that this belongs in stage 3.

### Seen (orch, 2026-10-17)

In a demo, `ly:orch` posted #3 to a fresh Claude strand `ly:demo`, whose body asked for a reply by `lyx creel post --reply`; the agent read it and replied as #4.
It replied by post only because the body told it how.
An inject (#5) asking for `PONG` was answered in the receiver's own pane, to the operator, and never reached the sender: an inject is typed as a user line and has no reply path.
`lyx creel read` prints the message with its id but says nothing about answering by `post --reply`.
Creel already limits inject: only EXTERNAL callers, programs and orch strands may inject into another strand (`lyx creel inject --help`).

### Operator view, not a decision (2026-10-17)

Creel needs a strict regime, so that an agent that sends something to another agent never ends up without an answer because the receiver wrote its answer in its own pane, to the operator.
Inject and post are two different things.
An inject looks as if the operator typed it, and that is its point.
A post takes an active step by the receiver to fetch it, and that step should tell the agent the message came from outside, not from the operator, so it cannot answer by writing in its pane.

### Orch view on that

The split holds, but the agent should be told rather than left to infer it: the reminder that starts its turn is itself typed into the pane like an operator line, and a model answers in its pane by default.
One line in `lyx creel read`'s output (the sender is another strand, not the operator; answer with `lyx creel post --reply <id> <text>`) makes it explicit at the moment it matters.

### Idea (orch recommendation)

- `lyx creel read` prints, with each message, the command that answers it (`lyx creel post --reply <id> <text>`), and says when the sender waits on the reply (`--wait-reply`).
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
- The journal gets an event per chase (`chased`, with the message id, receiver and chase number) and one `unanswered` event when the chases run out; `lyx creel log --id <id>` then shows a message's whole path, posted, read, chased, replied or unanswered.
- A report, beside `lyx report convergence`, counts expected replies, chases and unanswered messages per receiver role, so the stencils of roles that often need chasing can be tuned.

### Open

- The flag's name, and whether a reply by `defer` (as in the orch's note request) counts as an answer.
