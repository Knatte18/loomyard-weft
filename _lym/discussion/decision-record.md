# Decision record — creel-agents

## Goal

Creel stage 3: every agent reaches its parent, its seat peers and the orch through creel, never through Claude Code's `SendMessage`, so a non-Claude engine can take a role that has a parent.
A message that expects an answer is tracked until it gets one: creel chases a receiver that read it and did not reply, records the outcome, and a report shows which roles need chasing.

## Scope

In:

- A `--parent` receiver on `lym creel post` that resolves the asking agent's parent orch at post time.
- The hold notice's notifier wirings resolve its orch with the rule `--parent` uses (D3).
- The parent directive, the hold notice, the LLM driver's notify rules and every other stencil or Go message text that names `SendMessage` move to creel, per stencil-audit block 7 (`docs/research/stencil-engine-audit-2026-10-24/report.md`).
  This covers Claude-to-Claude messages too, so one route serves every engine.
- Chair–advisor messaging moves to creel; `RequirementPeerMessaging` goes.
- The appended creel trust sentence every Claude session gets is reworded to match the directives' trust rules.
- `lym shuttle send` and `lym shuttle interrupt` "move behind creel" by removal: creel's `post`, `inject` and `interrupt` replace them.
- Reply discipline: an expected-reply mark on a post, bounded chases, an `unanswered` outcome, their journal events, and a `lym report replies` verb.
- A scan test that keeps `SendMessage` out of stencils, prompt fixtures and Go message text except the in-session fork addressing a `ForkSubagents` role uses.
- `internal/creelengine/doc.go`'s "Unbuilt: the stage 3 remainder" section is replaced by the built design.

Already met, no work: the parent-review relay the creel doc lists for stage 3 is the Go driver's `parent_notice` relay, which already posts through creel (`internal/sheddriver/drivehandback.go`).

Out:

- A Codex engine and Codex hook delivery (note `codex-engine`); a Codex strand is a pane-mail strand and gets creel's reminder.
- Webster Merriam's message to its own fork by agent id: in-session fork addressing, Claude-only, already declared through `ForkSubagents`.
- Lym's own typing into a pane that is not a message between sessions: shuttle's launch prompt, skill load turns, resume lines, command injects, `lym loom resume`'s park line, and the seat engine's up, not-started and ending lines to the chair.
- Removing the reminder-and-read turn for pane-mail strands (note `creel-carry-agent-mail`).

## Decisions

### D1 — `lym creel post --parent` resolves the parent at post time

`lym creel post --parent <text>` takes no receiver argument; the client resolves the receiver from the caller's worktree with the rule batten's run notices use (`internal/battencli/notices.go`): from a pair, `orchcli.NotifyParent`'s walk, which resolves the pair's parent worktree with `hubgeom.ResolveParent` and walks to the nearest live orch from there; from a beam or the prime, `orchcli.NotifyFrom`'s walk, which starts at that worktree itself.
The success envelope names the resolved receiver.
It is refused, with the way forward (post to a strand by name, listed by `lym reed list`), when the resolved worktree records no orch strand.
`--parent` with a receiver argument or with `--reply` is refused before any call.

Rationale: the parent directive names the static parent orch, which on a beam may be dormant or never started, while hold and stop notices go to the nearest live orch; one rule for both makes an agent's question and its hold notice reach the same session, which is the risk the board entry asks this Discussion to check.
Bound: the flag adds no privilege; it only picks a receiver the caller could already name, and the daemon applies its usual sender and receiver rules to the post.

### D2 — The parent directive asks and trusts through creel

`contracts/stencils/parent/parent-directive-parent.md` says: ask your parent with `lym creel post --parent --expect-reply "<text>"`, then end your turn; the answer arrives as `[creel]` mail; answer a creel message with `lym creel post --reply <id> "<text>"`.
The trust sentence widens from `{{.parent_name}}` alone to `{{.parent_name}}` or any orch strand of this hub (role `orch`) as creel stamps the sender, since D1 can route a question to an orch further up the chain.
Mail from a lym program of this hub (an upper-case name such as `<shortname>:LOOM` or `<shortname>:CREEL`) is lym's own and is acted on; the LOOM decision-notice sentence stays as it is.

Rationale: one wording for every role with a parent, true on any engine that can run a shell command.
Bound: the widened trust reaches only orch strands, the operator's own delegates, and lym's programs, whose claim is bounded as below.
The sender stamp is as strong as creel's identity: on Linux it is read from the connecting process; off Linux it is the strand-name environment, which Claude sessions' shell guard refuses to set and a Codex session does not guard.
The same gap already applies to the `{{.parent_name}}` trust today; widening adds the other orch names to what an off-Linux Codex agent could pose as.
A program claim is accepted off Linux from any process whose strand name names no strand of this hub, and on Linux from an orch pane or any process outside every pane, through the Go client only, since the CLI has no program flag.
Lym verbs that post caller-supplied text as a program, such as `lym loom decision add` as `LOOM`, therefore reach agents as followable mail from whoever runs them; this is bounded by the 0700 lymtemp root that holds creel's socket and by the operator's user, the same boundary creel's doc states for program posts today.
Program trust is not narrowed to named programs: every program's mail is lym's own, and a named list would have to track each new program.

### D3 — The hold notice names creel and keeps its destination

`internal/shuttleengine/hold.go`'s notice says, in place of `SendMessage`: when the held agent's marked question is in your creel mail, answer it with `lym creel post --reply <id> "<text>"`; otherwise post to the strand with `lym creel post <strand> "<text>"`, ending the message with `shuttleengine.MessageTail`; its comments follow.
The notice keeps going through the runner's notifier, and every notifier wiring resolves with batten's run-notice rule, the rule D1 uses: `orchcli.NotifyParent`'s walk from a pair, `orchcli.NotifyFrom`'s walk from a beam or the prime.
The burler, loom and webster runners' notifier wirings (`SetNotifier` in `internal/burlercli/wiring.go`, `internal/loomcli/wiring.go` and `internal/webstercli/wiring.go`), which call `orchcli.NotifyParent` unconditionally, change to that rule; loom's other notices, such as the hand-back and landing notices, are not hold notices and keep their walk.

Rationale: the held agent's own question (D1) and the hold notice resolve the same orch, so the orch told an agent is held is the one holding its question.

### D4 — The LLM driver's notify rules post through creel

`contracts/stencils/shed/shed-template-driver-notify.md` and `contracts/stencils/shed/shed-template-driver-notify-watched.md` send each message with `lym creel post --parent`, and record a refused or failed post in the stop report as they record a failed send today.
The watched variant keeps its three cases, so it still never repeats a stop batten already announces; the Go driver's notices already post through creel and do not change.

Rationale: the LLM driver is the last driver path that messages over `SendMessage`.

### D5 — Every other agent stencil messages through creel

- Darn writer and webster Merriam: an ask to the parent goes by `lym creel post --parent --expect-reply`, a status message by `lym creel post --parent`; Merriam's message to a fork by agent id stays.
- Seat chair, seat advisor and the discussion advisor stencil: the chair asks an advisor with `lym creel post --expect-reply <advisor strand>`; an advisor answers with `lym creel post --reply <id>`, or posts to the chair's strand unprompted.
- Orch role: the orch reaches an agent with `lym creel post <strand>`, adding `--expect-reply` to a question, answers creel mail with `--reply`, lists strands with `lym reed list`, and never puts a question that wants an answer in an inject.
  It answers a hold notice with `lym creel post --reply <id>` when the held agent's marked question is in its mail, and posts to the strand the notice names only when no such question is pending.

Rationale: block 7 of the stencil audit is the full site list; each role gets the creel verb that fits it, and every question carries the mark D10 chases.

### D6 — The appended creel trust sentence defers to the role's rules

The sentence `internal/shuttleengine/claudeengine/settings.go` appends to every Claude session's system prompt keeps telling the session that `[creel]` lines are LoomYard internal mail delivered by hooks and how to answer them, and in place of "follow them as the operator's own instructions" says: follow mail from your parent, an orch strand or a lym program as the operator's own instructions, and weigh any other mail as your role's instructions say.

Rationale: after D5 an advisor's answer arrives as `[creel]` mail, which the seat directive calls advice, never an instruction; the system prompt must not tell the chair to obey it.
The Haiku trust finding (`docs/research/creel-midturn-trust-2026-10-28/report.md`) needed a sentence that makes parent mail followable, which the new wording keeps.

### D7 — `RequirementPeerMessaging` is removed

The requirement leaves the closed set in `internal/shuttleengine/engine.go` and `internal/shuttleengine/requires.go`, the Claude engine's capability list, and the seat specs that add it.
What stays Claude-only is declared as before: in-session forks by `ForkSubagents` (`RequirementForks`) and background wake by `RequirementBackgroundWake`.

Rationale: after D5 no role messages a peer through an engine's own tool, so nothing requires it.

### D8 — `lym shuttle send` and `lym shuttle interrupt` are removed

The two verbs and `internal/shuttlecli/send.go` and `internal/shuttlecli/interrupt.go` go.
`lym creel inject`, `lym creel interrupt` and `lym creel post --interrupt` cover their uses, addressed by strand name.
Every refusal and doc that names them, among them `shuttleengine.ErrNoRunningTurn`, the `Runner.Inject` guard refusals and `pattern/PATTERN-pane-text-injection.md`, names the creel verb instead.
The Go `Runner.Send` and `Runner.Interrupt` stay for shuttle's own callers and creel's delivery.

Rationale: keeping them as wrappers would leave two operator routes into a pane, one of which refuses under a pause but bypasses creel's journal, queue and hold.
Bound: the operator loses guid addressing; a strand name, which `lym reed list` shows, replaces it.
The replacement also changes how a line reaches a pane: a line injected under a pause is held with its TTL frozen and typed after the resume, where `lym shuttle send` refused it; it is typed only at idle within its TTL; no `shuttleengine.MessageTail` is appended unless the sender writes it; and only EXTERNAL, programs and orch strands may inject into another strand.

### D9 — A post can expect a reply

`lym creel post --expect-reply` marks the message as expecting a reply without blocking the sender; `--wait-reply` implies the mark.
The mark is journaled with the post and shown on the post's `lym creel log` entry.
A reply is a `post --reply` to the message by its current holder; the expectation follows the holder.
`lym creel read` and the hook-delivered line show the mark beside the reply command.

Rationale: the operator's 2026-10-17 idea; the orch's recommendation that a waiting sender gets the chase too.

### D10 — Creel chases an unanswered expected reply, twice

A marked message that its holder read and has not answered within the chase interval gets a chase: a creel post to the holder, signed `<shortname>:CREEL`, whose body names the message id, its sender and the reply command, and carries no text the sender wrote.
A chase and the sender notice name an EXTERNAL sender by the bare EXTERNAL name, without its envelope.
At most two chases go, one interval apart; one interval after the second, the message is `unanswered`: creel journals it, posts the sender a notice naming the message and the receiver when the sender is a strand or a program that has read once (otherwise the journal record alone stands), and answers a waiting `--wait-reply` with an `unanswered` outcome, which the CLI prints in its envelope and exits 1 on, and the Go client returns as a distinct outcome.
A reply that comes later is still delivered.
A chase cannot be forwarded: a forward of one is refused, naming the way forward, answer it with `lym creel post --reply <chase id>` or forward the chased message itself.
A `post --reply` to a chase is taken as a reply to the chased message, and is accepted only from that message's current holder; from a strand that no longer holds it, having forwarded it after the chase, it is refused, naming the way forward: the strand that holds the message now answers it, shown by `lym creel log --id <id>`.
The chase interval is a named constant in `internal/creelengine`, 15 minutes, like creel's other intervals.
The clock counts only while the holder is live, no pause covers it and no other sender's hold stands on it: it stops while the holder is dormant or paused, as an inject's TTL does, and also under another sender's hold, which an inject's TTL does not.
The consumed chase time is journaled the way an inject's consumed TTL is: a `chase-aged` record carrying the consumed seconds, written at the inject's persist interval (`agedPersistInterval`) while the clock runs and when it stops.
The removed-holder sweep's subjects, today strands holding unread mail or a queued inject (`sweepSubjects` in `internal/creelengine/sweep.go`), widen to strands holding a read, marked, unanswered message, and the sweep runs while any such strand exists.
A holder the sweep finds gone by its two-miss rule has each such message settled `unanswered` at once, with the reason `holder-gone`, and a waiting `--wait-reply` answered; no chase goes to it.
Chases ride the ordinary delivery: hooks for a hook strand, a typed notice for a pane-mail strand.
The chase number and the chased message's id travel on the chase's own `posted` record, which the restart counts, so a crash between the chase's post and its `chased` record never yields a third chase; the restart writes a missing `chased` record from that `posted` record.
The daemon derives the chase state from the journal (read time, `chase-aged` records, chase `posted` records), so a restart neither repeats nor drops a chase and counts no stopped time; after downtime, overdue steps go one at a time, each one interval after the later of the previous step and the restart.
Journal retention carries a read, marked, unanswered message forward like an unread one, with its consumed chase time.
A daemon timer per receiver holding a read, marked, unanswered message wakes at its next due step and never more than once a second.

Rationale: an agent that answers in its own pane leaves the sender without an answer; a fixed, content-free chase tells the receiver the reply route at the moment it matters.
Bound: at most two chase posts and one sender notice per marked message; a chase carries no sender text, so it cannot pose as an instruction.
A marked message never read is not chased.
A pane-mail holder gets the reminder backoff; a live hook-strand holder gets one ring and at most one bell per claimable set and nothing more; the removed-holder sweep covers either holder once it is gone.
A waiting `--wait-reply` sender on a message a live hook strand never reads waits until its `--timeout`, and without end when it passes none.

### D11 — Chases and outcomes are logged

The journal gets a `chased` event per chase (message id, receiver, chase number), `chase-aged` events carrying the consumed chase seconds (D10), and one `unanswered` event (message id, receiver, reason); `lym creel log --id <id>` shows a message's whole path, posted with its mark, read, chased, replied or unanswered.
A log entry carries the reply-to id of a reply alongside the mark, so `lym report replies` counts a reply against the marked message it answers.

Rationale: the operator asked that every chase be logged.

### D12 — `lym report replies`

A new `lym report replies` verb, beside `convergence` and `timing`, counts per receiver role the expected replies, the replies, the chases and the unanswered messages over the creel journal's seven-day retention (`journalRetention` in `internal/creelengine/journal.go`), read through creel's Go client log call.
It runs from the prime and changes nothing, like the other report verbs.

Rationale: the orch's recommendation, so the stencils of roles that often need chasing can be tuned.

### D13 — A scan keeps `SendMessage` out

A test fails on `SendMessage` in any file under `contracts/stencils/`, any prompt fixture under an `internal/**/testdata/` directory, or any Go string literal outside `_test.go` files, except an allowlisted site in a stencil whose role's spec sets `ForkSubagents`.
The webster Merriam fork-addressing line is the one allowlisted site; the seat smoke fixtures move to the D5 wording.

Rationale: the engine-neutral rule holds only while nothing re-adds the Claude tool by name.

### Added after Discussion (parent, 2026-10-28): The 2026-10-27 wait for a real mid-turn run of creel hook delivery is met

Decision: The run proceeds. creel-hook-post landed on 2026-10-28 and hook delivery has run in real sessions since: the hub orch received mail mid-turn under tool results and at turn ends, and a darn run's writer received parent decisions while it worked. Drop the Open risks line on the wait.

Rationale: Operator decision in chat, 2026-10-28, answering review finding F2 of Discussion round 1.

### Added after Discussion (parent, 2026-10-28): A holder that disappears after reading a marked message is found gone (review F1)

Decision: In D10, replace the sentence that relies on the removed-holder sweep with a stated mechanism: the sweep's subject set and its run condition widen to strands holding a read, marked, unanswered message (internal/creelengine/sweep.go sweepSubjects today takes only unread mail and queued injects), and the sweep's two-miss rule then settles each such message unanswered with reason holder-gone and answers a waiting --wait-reply. Add to D10's acceptance criterion a row where the holder holds no other mail or inject when it disappears after reading.

Rationale: Review finding F1 of Discussion round 1, taken as written; the operator chose to accept the round with its fixes recorded rather than run another round (2026-10-28).

## Constraints

- `PATTERN-parent-directive`: the directive stays the one place a spawned role learns how to ask; the orch stencils stay exempt.
- `PATTERN-agent-name`: D1's resolved receiver, D10's `CREEL` sender and D12's role grouping use `internal/agentname`.
- `PATTERN-daemon-wakeups`: the delivery loop's workers end when nothing is pending, so D10's chase timer is a wait of its own, bounded as D10 states, and the removed-holder sweep widened by D10 runs only while a strand holds a read, marked, unanswered message, with its existing backoff (`sweepBackoffBase` to `sweepBackoffCeiling`).
- `PATTERN-refusal-way-forward`: D1's refusal, D10's refusals of a chase forward and of a chase reply by a former holder, and every refusal re-pointed by D8 name the way forward, and a test reaches each.
- Creel keeps no config file (`internal/creelengine/doc.go`), so the chase interval is a constant, not a config key.
- `creelcli` cannot import `orchcli`, which imports it; D1's walk and its orch liveness check must be reachable from `creelcli` without that cycle.
  They can be: `go list -deps ./internal/creelcli` already includes `internal/orchengine`, `internal/hubgeom`, `internal/reedengine` and `internal/fabricengine`, which the walk and `orchcli`'s `OrchLive` and `OrchStrandOf` need, so they can move below `orchcli` without a new import edge.

## Auto-mode assumptions

- The operator's promotion of this task to running settles the 2026-10-15 "note until creel has proven itself" hold; the 2026-10-27 wait is raised under Open risks rather than assumed met.
- No creel `defer` verb exists, so the board's open question whether a deferral counts as an answer has nothing to decide.

## Open risks

- The board entry says the operator's 2026-10-27 wait (a real mid-turn run of creel's hook delivery) stands, and the `creel-hook-post` real-Claude scenarios did not run in the verify gates; the task is nevertheless running. Parent: is that wait met, or should this run hold?
- Moving the chair–advisor channel from `SendMessage` to hook-delivered creel mail changes when a chair sees an advisor's answer; a Discussion or Plan table with advisors is the first real run of it.

## Acceptance criteria

- D1: `lym creel post --parent` from a pair whose parent orch is live posts to it; from a pair whose parent is a beam with no live orch it posts to the nearest live orch above, the prime's at the end; from a beam it starts at the beam's own orch; where the resolved worktree records no orch strand it is refused naming the way forward; with a receiver argument or `--reply` it is refused before any call.
- D1, D3: for one held agent, in a pair and in a beam, the receiver its `--parent` post resolves and the orch its hold notice reaches are the same strand.
- D2: the rendered parent directive names `lym creel post --parent --expect-reply` and `--reply`, trusts any orch strand and lym program of the hub, and names no `SendMessage`.
- D3: the hold notice line names `lym creel post --reply <id>`, `lym creel post <strand>` and `shuttleengine.MessageTail` and no `SendMessage`.
- D3, D5: an orch that answers a hold notice with `lym creel post --reply` to the held agent's marked question clears the mark, and no chase follows.
- D4, D5: every stencil site of audit block 7 names a creel verb, every question site carries `--expect-reply`, and the webster fork-addressing line is unchanged.
- D6: the appended system-prompt sentence names parent, orch and program mail as followable and other mail as weighed by the role's rules.
- D7: no spec requires peer messaging, and `requires.go`'s closed set refuses the old `peer-messaging` name as unknown.
- D8: `lym shuttle send` and `lym shuttle interrupt` do not exist; `shuttleengine.ErrNoRunningTurn` and the `Runner.Inject` refusals name creel verbs.
- D9: `--expect-reply` and `--wait-reply` posts carry the mark; `read`, the hook line and the log entry show it; a reply by the holder clears it.
- D10: with an injected clock, a read and unanswered marked message gets a chase after one interval, a second after two, and is `unanswered` after three, with a sender notice and an `unanswered` answer to a waiting sender; a reply before any of these stops the rest; a reply to a chase by the holder answers the original, and by a strand that forwarded the chased message after the chase is refused naming the way forward; a forward of a chase is refused naming the way forward; a pause or another sender's hold stops the clock, and a restart after a pause counts no paused time; a holder gone after reading makes the message `unanswered` at once with reason `holder-gone`, also when the holder holds no other mail or inject when it disappears, and a waiting `--wait-reply` gets the `unanswered` outcome; a daemon restart between chases neither repeats nor skips one, a crash between a chase's post and its `chased` record yields no third chase, and after long downtime the overdue steps come one interval apart; retention carries a read, marked, unanswered message forward; a program sender that never read gets no notice and the journal records the outcome; a chase carries no sender text and names an EXTERNAL sender without its envelope.
- D11: `lym creel log --id` lists the mark, the `chased` and the `unanswered` events of a chased message.
- D11, D12: a reply's log entry carries the reply-to id of the marked message it answers, and `lym report replies` over a journal fixture counts expected, replied, chased and unanswered per receiver role, counting as replied only a reply paired with a marked message.
- D13: the scan fails on a stencil, testdata prompt or Go literal naming `SendMessage` outside the allowlist and passes on the tree after this task.

## PATTERN changes

- `PATTERN-agent-messaging` — adds: Messaging another session from an agent: only through creel, never an engine's own tool, bar a fork addressed in-session under ForkSubagents. (test)

## Notes for the plan writer

- `internal/orchengine/doc.go` notes about `SendMessage` across `/clear` restate for creel mail.
