# creel stage 1: the one way to reach an agent, standalone

`creel-core` · Task · feature, creel, reed, shuttle · running · Webster-Burler-Review

## This task: stage 1

Build stage 1 of the design below: creel alone, the daemon, the queues, the journal, every verb, and delivery through shuttle gated on the receiver being idle, tested against fake panes and in the tmux tier; no other module sends through it yet.
Stages 2 and 3 are the tasks `creel-notices` and `creel-agents`.
When this lands, the design moves into the creel package's `doc.go`, which those tasks then read.

## Operator decision (2026-10-15)

Creel is a prerequisite for Codex: agents reach their parent and each other through creel, not Claude Code's `SendMessage`.
Creel delivers through shuttle, so its idle check and send go through shuttle and work for any engine shuttle drives.

## Operator decision (2026-10-15, later): a working creel first, in parallel

Build a working creel now, in parallel with the other runs, and test it thoroughly before anything that works today is switched over to it.
This task therefore no longer waits on `engine-registry`: creel calls shuttle's present idle check and send, and moves onto the engine registry when that lands.
Stages 2 and 3 (`creel-notices`, `creel-agents`) replace working paths, so they start only on the operator's word once creel has proven itself, never on a newly-ready auto-start.

## Design

Creel is the one module anything uses to reach an agent: a Go program, another agent or the operator in a terminal.
Every agent in lyx runs through shuttle and lives in a reed strand, so every agent has an address (`agent-naming`'s `<shortname>:<slug>:<role>`).
The decisions below are the operator's of 2026-10-06 and govern; the 2026-09-20 design under History is superseded where it differs.

## Why

Claude's `SendMessage` lands straight in the orch session, past the orch watcher, so nothing holds it during a context cycle.
A message that arrives while the orch writes its note is seen, but the note request says to write and end the turn, so it is lost at `/clear` unless the note happens to carry it.
A message that arrives after the note is written starts a turn whose end opens the clear gate, so that turn's work never reaches the note.
Batten notices do not have this problem: they go through the watcher's notice queue (`orchengine.QueueNotice`, `.lyx/orch/notices/`), which holds them during a cycle.
Every message goes through creel instead, agent to agent and module to agent alike, and the watcher's notice queue folds into creel.

## Decisions (operator, 2026-10-06)

### Verbs: one module, separate verbs per guarantee

- `lyx creel post <receiver> "<text>"`: mail. Stored in the receiver's mailbox; the receiver is reminded until it reads it.
- `lyx creel inject <receiver> "<text>"`: typed straight into the receiver's pane as its next input, once the receiver is idle. For commands (`/clear`, `/compact`, the reload) and creel's own reminder lines; anything the receiver must read and answer is `post`, not `inject`. No read receipt. Only privileged senders may inject into another strand; injecting into one's own strand is free.
- `lyx creel inject --interrupt <receiver> "<text>"`: interrupt the receiver's running turn, wait until it is idle, then type the text: "stop, and do this instead".
- `lyx creel interrupt <receiver>`: stop the receiver's running turn without killing its pane. It bypasses the queue, since it must act while the receiver is busy.
- `lyx creel forward <message-id> <receiver>`: move an unread message, envelope and body, to another receiver's mailbox; nothing stays behind in the forwarder's. The id is creel's, shown by `inbox` and in the reminder line (such as `#41`). Its `from` stays the original sender and the envelope's `path` gains the forwarder, so the message records the whole way it has taken (sender, then each forwarder in order) and the new receiver replies to the sender, not to whoever forwarded it. The forwarder need not read the body: the reminder line carries each message's sender and a short subject (its first line), enough to route. Creel knows nothing of why a message is forwarded (see `run-message-routing`, where a run's driver routes instructions to the agent working in its run).
- `lyx creel read`, `lyx creel inbox`, `lyx creel log`: below.

The operator deals with creel only.
No other module offers a verb that types into an agent's pane: `lyx shuttle send` and `lyx shuttle interrupt` move behind creel when it is wired (stage 2 or 3).

### Layering

`creel → shuttle → reed → tmux`, each layer owning one thing:

- **creel** decides what is delivered and when: the queue per receiver, priority (an inject goes ahead of mail reminders), holding, the journal and read status.
- **shuttle** knows the LLM: whether its session is idle (`IdleProbe`), and how text and keys reach it safely (paste, submit, the interrupt key). Creel asks shuttle and owns no idle probe itself. Shuttle knows nothing of queues or senders and exposes no send verb.
- **reed** knows which pane a strand lives in and does the keystrokes (`SendText` for literal text, `SendKey` for named keys such as Escape).

`PATTERN-orch-pane-single-writer` becomes: only creel decides what is delivered into an agent's pane.

### Mail is pulled

- `post` puts the message in the receiver's mailbox, and creel injects one short line, such as `[creel] 2 unread from ly:x:driver, ly:batten`.
- The line repeats each time the receiver goes idle, until it runs `lyx creel read`; only then is the message marked read. So "delivered" means "read", and creel knows it for certain.
- A busy receiver need not read at once: it knows it has mail and picks it up when it is done. An orch close to a context cycle leaves its mail unread, and creel reminds it after the cycle.
- No extra turn: the reminder starts a turn and the agent reads within it.
- A long message is a file the sender writes, and the mail says which file to read.

### Reading

- `read`: the oldest unread message, together with every other unread message from the same sender.
- `read --from <sender>`: every unread message from that sender.
- `read --all`: every unread message.
- `inbox`: the senders with their unread counts.

### Holding

A sender may hold mail reminders to a receiver and release them later.
The orch watcher holds `ly:orch`'s reminders during a context cycle, while its own messages still pass: the note request (post) and `/clear`, `/compact` and the reload (inject).

A pause from `lyx-halt` (`lyx pause`, hub-wide or per run) holds every delivery into a paused agent's pane, injects and reminders alike, until the pause is lifted; `post` still stores the mail (operator, 2026-10-12, from `lyx-halt`'s parent review).

### The note request is mail (operator, 2026-10-07)

The watcher posts the note request to `ly:orch` rather than injecting it, so the orch reads it when it is ready and is never interrupted mid-work.
The orch replies by post: `written <path>` once the note is written, or `defer`.
An injected request has no receipt: on 2026-10-07 the operator dismissed one at the prompt, the watcher counted it as delivered and waited silently until its timeout, and later `distill` requests queued behind it.
With post the watcher's timeout tells two cases apart:

- Unread: the orch has not seen the request; creel keeps reminding it at each idle.
- Read but unanswered: the orch saw it and did not reply; the watcher gives up and shows the reason in `lyx orch status` and reed.

A dismissed request is answered with `defer`, never left unanswered.

### Receivers

- An unknown name is refused, listing the names that exist.
- A known receiver that is dead or dormant keeps its queue, and delivery resumes when its strand is live again.
- A receiver whose strand is removed (a producer's agent ending, for one) leaves no mail behind unread: each unread message goes one step back along its `path` (operator, 2026-10-07): to the last forwarder, or to the sender when it was never forwarded, marked with the receiver that ended without reading it, so it can be routed again. When that step is gone too, it goes on up the path towards the sender; when the sender is gone or external, the message is marked undelivered and stays in the journal. Creel learns only that the strand no longer exists, never why: the daemon sweeps the pool (below) on each tick for unread messages whose holder is no longer a strand, so no event is needed when a strand dies.
- A terminal is never typed into: what is typed into a shell is a command. A terminal or program that wants an answer waits on creel's socket instead (`post --wait-reply`, `read --wait`) and prints it.

### Daemon and transport

- A creel daemon of its own per hub, apart from reed's watchdog, since reed must not depend on creel. It owns the queues, the reminders and the read status.
- It listens on a Unix socket named by the hub's shortname, such as `$XDG_RUNTIME_DIR/lyx/creel-<shortname>.sock`, acknowledges receipt, and is started by any creel verb that finds it down, as `lyx reed` starts its tmux server.
- The sender is named from the environment: `LYX_STRAND_NAME` for a strand in reed, and a name the caller passes for a Go program.
  Anything else (a terminal, or a Claude session started outside reed) is `<shortname>:EXTERNAL`, such as `ly:EXTERNAL`: upper case, so it never collides with a strand name, and it claims only what creel knows, that the sender is not a strand.
  The envelope adds what creel can see of an external sender: its terminal, the process that ran the verb (`claude`, `bash`) and its directory, shown as `from ly:EXTERNAL (claude, pts/4, ~/Code/…)`.
  An external sender has no address, so a reply reaches it only through `--wait-reply`.
- A Claude session the operator adds to reed by hand (`lyx reed add --name explore --cmd claude`) is a strand with an address, though shuttle did not start it; shuttle's idle check must work on such a pane too, and stage 1 tests it.
- A filesystem drop-box (sender writes a file into the receiver's queue directory) was set aside: several writers in one directory, and no receipt for the sender.

### Storage

- One pool per hub, not a mailbox per receiver (operator, 2026-10-07): every message is stored once with its id, its sender and its current holder, so `forward` only changes the holder, and a sweep finds every message whose holder is gone.
- Access follows the holder: `read` and `forward` work only on messages the caller holds, so "read" always means the holder read it and no agent reads another's mail by mistake or by guessing an id. A sender may see the status of its own messages (read, by whom), never take them back.
- JSONL files under the hub's `.lyx/creel/`, written only by the daemon.
- Unread messages stay until read; read messages and journal lines are deleted after 7 days, so a week of traffic stays available for debugging.
- The journal holds one line per event (posted, reminded, read, injected, interrupted), and `lyx creel log` shows it.

### Later, not stage 1

- Sending across hubs is a feature of its own. The address already picks the hub's socket, so it costs nothing in stage 1.
- An MCP layer on top of the verbs, if an agent outside a terminal ever needs one.

## Build in stages (operator, 2026-10-06)

Creel is a new module that every delivery will depend on, so it is built and tested standalone before anything is wired to it.
Each stage below becomes its own task when creel is promoted.

1. Creel alone: the daemon, the queues, the journal, every verb above, and delivery through shuttle gated on the receiver being idle, tested against fake panes and in the tmux tier. No other module sends through it yet; the operator tries it from a terminal.
2. System notices: the orch watcher's and batten's notices and the watcher's injects go through creel, and the watcher's notice queue and idle probe go. The watcher keeps only its cycle logic (tracking tokens, choosing `refresh`/`distill`/automatic cycles, waiting for the note, sequencing the reload, its state and lifecycle), and advances a cycle phase only on the orch's reply to the note request, or once creel confirms a command inject was typed.
3. Agents: the parent-review gate (`internal/parentreview/gate.go`, which today prompts Discussion-Write to `SendMessage` the parent) posts itself, drivers' stop reports post instead of duplicating batten's notices over `SendMessage`, and `lyx shuttle send`/`interrupt` move behind creel.

Fixes that do not need creel, such as `shuttle-asking-holds`, go ahead of it.

## History: the 2026-09-20 design (superseded where the decisions differ)

Deferred 2026-10-02 while Claude Code's `SendMessage` was enough; resumed 2026-10-06 for the reason under Why.
Its address grammar (`name@slug[.hub]`) is superseded by `agent-naming`'s `<shortname>:<slug>:<role>`, and its pane-title and session-name reconcile belongs to reed.

### The idea

Deliver messages/events to any Strand (agent, operator, `ly-drive` watchdog, orchestrator) by address.

### The send/receive asymmetry

Being a Strand is only required to *receive* mail — an address needs somewhere durable to deliver to. It is **not** required to *send*: anything able to reach the delivery mechanism can send without needing an address of its own. An in-process Claude Code fork subagent is the clearest example — it can never itself become a Strand (it is never an OS/tmux process), but it can still send.

### Addressing

The address format is **`name@slug[.hub]`** — the email shape, deliberately: "deliver to *name* at *place*" needs no explanation for humans or agents, and the grammar is collision-free because slugs (`[a-z][a-z0-9-]*`) can contain neither `@` nor `.`, so the single `@` splits unambiguously and the optional `.hub` suffix reads exactly like an unqualified-vs-qualified hostname (omitted = own hub).
The resemblance to real email is a known confusion risk, accepted on purpose — it buys the mental model.
Real external channels never ride on it: a raw email address is never a valid recipient, and creel never guesses whether something "is" email.
Instead — **later, not creel v1** — external channels are **connector domains**: the rule is that the rightmost element decides the router — a known hub (or nothing) means internal post, while a reserved connector domain (`slack`, `email`, …) hands *everything left of it* to that connector module to interpret (`user@slack`; possibly `orch@slug.hub.slack` if each slug gets its own Slack channel — the connector owns that mapping, creel's core never parses it).
Each connector module owns its own list of valid left-side names, and that configured list *is* the allowlist: a confused agent structurally cannot reach an arbitrary third party.
Inbound goes the same way: a reply arriving from Slack enters creel as ordinary mail with the connector address as `from`, so the receiving agent just replies to `from` and never knows Slack exists.
Connector domains are excluded from the hub-name grammar, so no hub can ever shadow one.
Resolution stays heuristic-free everywhere: a recipient resolves to a strand, or ends in a connector domain, or is refused with the list of names that exist.
Creel v1 reserves the grammar only; no connector ships with it.
(The `@` file-mention in Claude Code's interactive input is a UI autocomplete trigger only — an address in a shell argument is just text — so the overlap costs a cosmetic popup when a human types an address in the prompt box, nothing more.)

- **The strand guid is the identity; names are the address.**
  Every strand has a guid for its whole life, pane or no pane; creel resolves an address at *delivery* time (slug → session, name → strand → guid) and delivers to the guid.
  Names are what humans and agents write, because names are role-stable while guids are instance-bound:
  a torn-down-and-respawned orchestrator has a new guid but the same name, and mail to `orchestrator@task-a` should reach whoever holds the role now.
  A reply can carry the guid when pinning one specific instance matters (detecting "the one I was talking to is gone").
- **Self-identification is injected, not looked up.**
  Reed owns every spawn/relaunch command line, so it injects `LYX_REED_ID` (the guid) and `LYX_STRAND_NAME` into every process it starts.
  The full address is *derived*, never injected: worktree slug comes from the cwd resolution the process already lives under, so `<slug>:<$LYX_STRAND_NAME>` composes at the point of use — a third stored copy could only drift.
  Fallback and verification: `$TMUX_PANE` (the `%`-id, server-unique, never renumbered) joins against the state's `PaneID` field; env and join disagreeing is an anomaly worth surfacing, not just a fallback order.
  Agents should treat the injected name as their own: ly-drive and sibling stencils read `$LYX_STRAND_NAME` at session start and use it in self-reference and as sender.
- **Names mirror to pane titles for humans, authority stays in state.**
  Reed sets the tmux pane title (`select-pane -T`) to the strand name at spawn/relaunch; tmux titles are display-only and enforce nothing, so the state's name field remains the single lookup key — the same volatile-view/durable-truth split `PaneID` already has.
  Reed owns the title channel alone: `allow-set-title off` on reed-managed panes, because any program in a pane can otherwise retitle it via the OSC 0/2 escape at any time (Claude Code does, continuously), and a cooperating-agent scheme is exactly the drift vector to close, not a feature.
  The per-hub daemon's tick adds the safety net: compare each tracked pane's actual title against the strand name, rewrite on divergence, and log that it happened — reconcile, same as the rest of reed's repair machinery.
  And the hard rule that makes display drift harmless rather than merely unlikely: **delivery never resolves through a pane title** — routing is always name → state → guid, so a stale title can mislead an eye but never a message; a send to a name the state doesn't know refuses with the list of names that exist (the same refuse-with-list idiom shed's run addressing uses), which itself surfaces the drift.
  A listing verb (`lyx reed list`: name, guid, session/worktree, pane-id, actual pane title, alive/dormant, drift flagged) doubles as the address directory and the diagnostic for exactly this concern.
- **An agent's own session name is the strand name too, and the daemon owns it.**
  A Claude process in a strand carries a session name of its own (`claude --name`, `/rename`) — the name the operator sees in the session list and that session-to-session messaging addresses.
  It must equal the strand name, for the same reason the pane title must: two names set by different hands drift apart and go stale.
  Setting it once at spawn is not enough, because a Claude session's name is reset after a compact, so a long-running agent silently loses it mid-run.
  So the per-hub daemon's tick owns it as it owns the title: read each tracked agent strand's current session name, and when it differs from the strand name, restore it (e.g. `/rename <strand name>` sent into the pane at idle, the same idle-only delivery rule as mail), logging the repair.
  The strand name stays the single source of truth; the session name, like the pane title, is a mirror reed keeps in sync, never a key anything resolves through.
  "Every pane has a name" is then nearly free: everything reed spawns is a strand (the operator's terminal is Selvage, which is not a strand and so not addressable), every strand has a name, every name mirrors to its title; panes outside reed are outside the system and never routed to.
- **Rename is a relaunch.**
  Env is frozen at process start, so the name is a birth attribute set at `AddStrand` and never mutated in place — renaming a role means tearing down and respawning its strand.
- **Well-known role names make addresses guessable.**
  A small canonical vocabulary (`orchestrator`, `driver`, `operator`, …) lets an agent address `driver@<slug>` with no directory lookup at all; a listing verb covers the rest.
- **Multi-hub is out of scope, but the grammar reserves the door.**
  Operating across several repos at once would not need one reed server spanning hubs: each hub keeps its own server and per-hub daemon, and cross-hub mail becomes daemon-to-daemon relay — the optional `hub:` prefix (omitted = own hub) is the entire forward-compatibility cost, paid here in grammar only.

### Dependencies

Deliberately a separate, later task from `ly-drive + orchestrator`'s launch-convention change and the shipped `reed: born-as-strand` item, which it depends on **for the receive side only**: everything that should be addressable must already exist as a Strand before addressing it means anything. Also downstream of `worktree spawn/teardown as Shed producers`' headless-orchestration future, since a mailbox is most useful once external actors can reach a worktree without a human ever having opened it first.

### Delivery

Transport and log are two different jobs, and the terminal is bad at the second one (scrollback dies with the pane, is unsearchable, and mixes mail into everything else) — so delivery is **pull, not full-content injection**, with a durable store carrying the audit trail.

- **Every message is an envelope in the creel store**: `from`, `to`, timestamp, `kind`, status (`sent` → `delivered` → `read`).
  `from` is filled in by the send verb itself, never free-typed: derived from `$LYX_STRAND_NAME` + the cwd slug when it can be (env is inherited down the whole process tree, so Go code running inside a strand signs with the strand's address for free), falling back to the `$TMUX_PANE`-to-state join.
  Outside reed there is no address, and creel does not pretend otherwise: `from` becomes a reserved not-addressable marker (`operator`), meaning a reply is impossible and the message must stand on its own — no reply-routing is configured for the marker.
  So a receiver always knows exactly where the reply goes (or that it can't), and a sender can neither forge nor forget its identity.
  Replying is just sending to `from`.
- **Two kinds.**
  `mail` is stored and *pulled*: the receiver is notified with one short line and reads at its own pace (`lyx creel inbox` for the count, a get-one verb that flips status to `read`).
  Pulled content arrives as tool output the agent asked for — better context than a giant injected user-message — and a short fixed notification line can never break on escaping the way multi-line `send-keys` can.
  `inject` is typed verbatim into the receiving pane instead: its body is a command to *execute*, not content to read, so it never sits in an inbox — the first use case is `lyx reed self-compact`, sugar for sending `/compact` to one's own address.
  Self-injection is free; injecting into *another* strand's pane is the driver privilege, restricted to privileged senders, never something any strand can do to any other.
- **The daemon delivers, at idle.**
  Both kinds ride the per-hub daemon's existing per-tick loop (resolving the old open item — no new process): the notification line or injected body is sent-keys into the receiving pane only when the receiver is idle, not mid-turn.
- **Everything is journaled, regardless of kind.**
  An `inject` is not mail but is still logged with sender, receiver, body, and delivery time; `lyx creel log` shows all traffic hub-wide — who sent, who read, when — searchable and durable, which is strictly better backtracking than terminal scrollback ever was.
  The live-watching human still sees traffic as it happens: the notification line carries sender + subject (`2 new: watchdog@prime "loom stuck at Gate"`), bodies stay in the store.
- **Plain CLI verbs, no MCP layer.**
  Creel needs no streaming or client-side session state, so it stays ordinary `lyx` verbs like every other module; an MCP server could be layered *on top of* the same verbs later if an agent outside a terminal ever needs access — deferred until someone does, not decided against.

## Related

- the shipped `reed: born-as-strand` item — the receive-side prerequisite.
- `internal/reedengine`'s package documentation — reed's watchdog daemon, which creel's daemon stays apart from.
- `shuttle-asking-holds` — the false asking stop, fixed ahead of creel.
