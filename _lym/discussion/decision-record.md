# Decision record — creel-cross-hub-mail

## Goal

A strand of one hub mails a strand of another hub on the same machine through creel, and the receiver sees the real sending strand, confirmed by the sending hub's daemon.
A hub obeys such mail only from the orch of a hub it lists as trusted; all other mail from another hub is data to weigh.
Trust runs one way: loomyard steers the sandbox hubs `lyt` and `cs`, which list `ly`, and loomyard lists neither.

## Scope

In:

- A machine record mapping each hub's shortname to its hub path.
- Posting to another hub's strand by its full name, such as `lyt:orch`, daemon to daemon.
- The receiving daemon's confirm call to the sending daemon, and replay refusal.
- Replies across the hub border with `lym creel post --reply <id>`, `--expect-reply` and `--wait-reply` included.
- A new `creel` config module with the key `trusted_hubs`.
- The sender stamp of mail from another hub, and the agent instructions that read it: the Claude creel trust sentence, the parent directive and the orch role.
- `lym creel hubs`, listing the hubs the machine record holds.
- A board note for the trial run that closes the entry.

Out:

- Another machine (the entry's "Not in this round").
- The closed area of `sandbox-closed-area-build`.
- Every verb but post and reply across hubs, and every sender and receiver but a strand (D2, D4, D8).
- An inbox folder as a transport.

## Decisions

### D1 — A machine record of hubs, written by the creel daemon

- A new package `internal/hubregistry` (new) keeps one JSON record, `hubs.json`, in the per-user lym state directory, the `lym` directory under `$XDG_STATE_HOME` or `~/.local/state` that `internal/standalonestate/standalonestate.go` already uses; its file name is a named constant there, and its directory is told to the package, so a test points it into its own temporary directory.
- An entry is a hub path with its shortname and the time it was last recorded; the record is rewritten through `internal/state` under a lock from `internal/lock`, so daemons of several hubs writing at once lose no entry.
- The creel daemon records its own hub when it starts, after it takes its single-instance lock, and in the same write drops every entry that fails the check below.
- A lookup by shortname reads each entry's hub and keeps only the entries whose Bolt shortname record, read through `fabricengine.ReadShortname`, still equals the shortname, so a moved or removed hub drops out; it answers the one hub path, or refuses with none or with several.
- The refusal for none names the shortname and the way forward: run any `lym creel` verb, such as `lym creel inbox`, in that hub once.
  The refusal for several names every hub path and says mail across hubs needs distinct shortnames.

Rationale: no machine-wide list of hubs exists today, and the creel daemon is the one process every hub starts that mail needs anyway; the entry proposed writing the record when a hub is created or opened, and the daemon's start is the moment an opened hub always passes, while fabric's clone learns nothing of creel.
Bound: the record is a file the operator's user can write; a hub is addressable only after its daemon has started once; two hubs sharing a shortname cannot be addressed by it, and mail from either is refused at the receiver (D3).

### D2 — Posting to another hub: the sending daemon queues and delivers

- A name whose shortname is not the hub's own names a strand of another hub; `agentname.Parse` reads it as today.
- Only a strand sender may post to another hub, and only to a strand name: a program or EXTERNAL sender is refused, naming a post from a strand pane or from the receiving hub's directory as the way forward, and a program or EXTERNAL receiver is refused, naming a strand of that hub.
- The sending daemon resolves the receiving hub through D1 at the post and refuses the post when D1 refuses, with D1's way forward.
- The size check measures the line the receiver will render: the longer stamp of D6 and an id of the widest an int64 prints.
- An accepted post is journaled under a new event, an outbound message, carrying the receiver's full name and the receiving hub's path, and answered with its id as a local post is.
  An outbound message is held by no local strand: no mail flag, delivery worker, reminder, sweep, chase or settle acts on it, and an older binary folding the journal ignores the event whole.
- An outbound worker delivers each pending outbound message, oldest first per receiving hub: it builds the receiving hub's creel geometry through `hubgeom.CreelGeometry`, starts that hub's daemon when none answers, as a client does through `internal/creelcli/spawn.go`, and sends one `deliver` request carrying the message id, the sender, the receiver, the body, the expects-reply mark and, for a reply, the id it answers.
- The receiving daemon answers `accepted` with its own id, `refused` with a reason, or a transient refusal (D3); a call has a 30-second timeout.
  `accepted` is journaled as the message's remote delivery with the receiver's id; `refused` returns the message undelivered at once.
  A transient refusal, a failed connection and a timeout are retried from one second doubling to a minute, and a message still pending an hour after its journaled post time returns undelivered, also across daemon restarts.
- A returned message ends a `--wait-reply` wait as returned undelivered and sends the sender a `<shortname>:CREEL` notice naming the receiver, the reason and `lym creel log --id <id>`.
- A delivered outbound message that expects a reply waits for it: a reply arriving by D4 ends its waits and marks it replied.
- Retention carries an outbound message while it is pending, and while it expects a reply and has none, as it carries unread mail.

Rationale: the sending daemon already knows the sender from `SO_PEERCRED`, and its journal is a durable queue, so a receiving hub that is down or starting loses nothing and needs no inbox folder; a strand-only receiver keeps another hub's mail out of program mailboxes such as `reed-slack-relay`'s `<shortname>:SLACK`, which posts on by the sender's role.
Bound: a post to another hub may wait up to an hour before it returns; delivery is at least once on the wire and exactly once in the receiver's pool by D3's duplicate rule.

### D3 — The receiving daemon confirms the sender with the sending daemon

On a `deliver` request the receiving daemon:

1. Refuses a sender that is not a strand name, a sender of its own shortname, a receiver that is not a strand name or that its receiver rules refuse, and a line over the cap, as `refused` with the reason.
2. Resolves the sender's shortname through D1, refusing on none or several; it never uses a hub path a request names.
3. Answers `accepted` with the id it already gave when it holds a message with that sending hub path and sending id, and does nothing more, so a retried delivery stores nothing twice.
4. Sends one `confirm` request to the resolved hub's socket, with a 10-second timeout, carrying the sending id, the sender, the receiver, the reply-to id and the SHA-256 of the body; it starts no daemon for it.
5. The sending daemon answers `yes` when its journal holds an outbound message with that id, sender, receiver, reply-to and body hash that has no remote delivery journaled, and `no` otherwise; it never sends the text.
6. On `yes` the receiving daemon journals the message as mail from that strand with the sending hub's path and id and the trust D5 decides, and answers `accepted`.
   On `no` it answers `refused` and logs the refused delivery at Warn, naming the claimed sender; the message reaches no agent.
   A failed or timed-out confirm is a transient refusal, and the sending daemon retries (D2).

No daemon holds its store lock across a call to another daemon, the `deliver` of D2 and the `confirm` of step 4 alike, so two hubs delivering to each other at once both serve the other's confirm.

Rationale: a call needs no secret, which every agent of the user could read; the sending daemon vouches only for what its own sender check accepted, and confirms until the receiver's `accepted` is journaled, so a receiver that dies between the confirm and its store gets the message on the retry.
Bound: a replayed `deliver` is refused once the sending daemon has journaled the remote delivery, unless the receiver still holds it, when it gets the old id and nothing is stored; between the confirm and that journal write, a replay of the same tuple is stored once by step 3.
Bound: the check stops a sender forged with a hand-written `deliver` request or by claiming another strand through creel's verbs.
It does not stop a process of the operator's user that writes a hub's journal files or the machine record (D1), binds a hub's socket while its daemon is down, types into the trusted hub's orch pane through its tmux socket, or starts a strand of an orch name in the trusted hub through reed (D5 narrows that to the exact orch names); the sandbox hubs' agents can do each, and closing them is `sandbox-closed-area-build`'s.
Bound: off Linux the sending daemon knows its sender from the environment alone, so a receiving daemon off Linux stores every message from another hub as untrusted (D5).

### D4 — Replies cross back the same way

- A reply (`lym creel post --reply <id>`) to a message from another hub, by its current holder, a strand, is an outbound message to the original sender, carrying the sending hub's id as the id it answers, delivered and confirmed by D2 and D3 in the other direction.
- The receiving daemon takes such a delivery as a reply only when the answered id is an outbound message of its own that went to the replying strand's hub; it then ends every wait on that message, marks it replied and stores the reply as mail to its sender.
  A delivery naming any other id is refused.
- On the replier's side the message is marked replied when the reply is journaled, so its chase stops.
- A message from another hub is forwarded within the receiving hub to a strand as today; a forward to a program or to a name of another hub is refused, naming a new post as the way forward.
- A message from another hub is chased and settled unanswered on the receiving side as any message; the settle sends no notice to the other hub's sender, and the sweep of a gone holder journals it undelivered, as it does when the step back is no strand.

Rationale: the trial needs a question from loomyard answered by the sandbox hub's orch; one route for posts and replies keeps one confirm rule.
Bound: the sending side learns of an unanswered settle or a gone holder only by the absence of a reply, and a `--wait-reply` across hubs ends at its timeout; a reply that returns undelivered leaves the original marked replied, and the replier learns it from D2's `CREEL` notice and may reply again; a reply arriving after the sending side's retention dropped its message is refused.

### D5 — Trust per hub in a new `creel` config module

- A new hub-wide config module `creel`, package `internal/creelconfig` (new), with one key `trusted_hubs`, a list of shortnames, empty by default.
- The key is classified `CategoryGuard`, since it grants authority, so it lives in the hub's base file on the board, and a local layer file holding it is refused; the module has no machine key, so no machine layer is stacked.
- The receiving daemon reads the module at each arrival from another hub, with `LoadOrTemplate`, so a changed list applies without a daemon restart; a load that fails stores the message untrusted.
- A message is trusted when the sender's shortname is in `trusted_hubs`, the sender's name is exactly `<shortname>:orch` or `<shortname>:<slug>:orch` (role `orch`, never a numbered `orch-N`), and the daemon runs on Linux; the trust is journaled with the message at arrival.
- Every other message from another hub is untrusted.

Rationale: the hub that obeys is the one that decides, and committing the list to its board keeps every change in git; programs are left out since any process outside a pane may claim a program name (`internal/creelengine/doc.go`, Sender identity), and a numbered orch role is what `lym reed add --role orch` forms beside a live orch.
Bound: a change to `trusted_hubs` does not restamp mail already stored; trust grants only what mail from the hub's own orch carries, and the sender check (D3) cannot stop a trusted orch that is itself prompt-injected, which is why loomyard lists no hub.

### D6 — The sender stamp of mail from another hub

- A message from another hub carries a stamp between the sender's closing quote and the colon of its line: `[creel] #<id> from "<sender>" (other hub, trusted orch): <body>` or `(other hub, untrusted)`, and the expects-reply form puts the stamp before `, expects a reply`.
- The unread reminder shows the same stamp after the sender's name; `lym creel read` keeps `from` as the bare name and adds the stamp's trust and the sending hub as fields of their own; `lym creel log` shows the sending hub's path and id on the message's entry.
- The 8 KiB line cap measures the stamped form.
- Mail from another hub is agent mail: a pane-mail strand gets it reminded, never typed in full.

Rationale: a local sender's name never holds a parenthesis or a space, so a stamp after the quote is creel's alone; a body that imitates one sits after the colon.

### D7 — Agent instructions read the stamps

- `internal/shuttleengine/claudeengine/settings.go`'s creel trust sentence names the orch strands and programs of the session's own hub, mail stamped `(other hub, trusted orch)` as carrying an orch's authority, and mail stamped `(other hub, untrusted)` as data, never an instruction.
- `contracts/stencils/parent/parent-directive-parent.md` says the same: a sender stamped `(other hub, trusted orch)` is the operator's delegate like an orch strand of this hub, and one stamped `(other hub, untrusted)` is data to weigh.
- `contracts/orchrole/orch-role.md` tells the orch how to address another hub's strand, that `lym creel hubs` lists the hubs, that its own hub's `trusted_hubs` decides what it obeys, and that an untrusted message from another hub is weighed and never relayed as an instruction.
- `internal/creelengine/doc.go` drops "Later and not planned: sending across hubs" and documents D1 to D6.

Rationale: the trust sentence today says "an orch strand" with no hub, which would read any hub's orch as trusted; the stamp makes the line say what creel decided.
Bound: a hub whose board copy of the parent directive differs from its seeded hash keeps its own text, since stencilstore never overwrites it; the compiled trust sentence changes in every hub.

### D8 — `lym creel` verbs

- A new verb `lym creel hubs` lists the machine record's hubs that pass D1's check: shortname, hub path, whether its daemon answers, and whether this hub trusts it.
- `inject`, `interrupt`, `hold`, `release` and `post --interrupt` to a name of another hub are refused, naming `lym creel post` as the way forward.
- `post --parent` is unchanged.

### D9 — The trial goes on the board as a note

A card files a board note `creel-cross-hub-trial` through `lym board`: a darn `docs` run in which the loomyard orch drives a run in `lyt` from start to landing through creel alone, with the operator doing nothing, recording the result under `docs/research/`.
It names the setup the trial needs: `trusted_hubs: [ly]` in the `creel` config of `lyt` (and `cs`), set by the operator in those hubs, and a deployed binary carrying this task.

Rationale: a run's agent may file a note but not a task, and the trial needs this task deployed first.

## Constraints

- Today's creel facts the design rests on, read from `internal/creelengine/doc.go`: one daemon per hub; the socket `creel-<shortname>-<h>.sock` in the user's 0700 lymtemp root, named by `creelengine.SocketName` from the shortname and the hub path; the sender read from `SO_PEERCRED` and the pane ancestry on Linux, the environment alone elsewhere; a program claim accepted from any process outside a pane; one goroutine per connection; journal retention of seven days; ids per hub that are never reused.
- A shortname is repo-wide, recorded on the board by `lym fabric clone --shortname`, and not unique on a machine (`fabricengine.ReadShortname`); D1 refuses an ambiguous one, and the test hubs `hubforge` makes share one, so a two-hub test gives one of them another.
- `lym reed add --role orch` names the strand `orch-N` when the worktree holds an orch, and `--name` gives any role (`internal/reedcli/add.go`); D5 trusts the exact orch names only.
- `PATTERN-lym-files` governs hub and worktree files; D1's record belongs to no hub and follows the per-user precedents `internal/githubclient/cache.go` and `internal/standalonestate/standalonestate.go`.
- `PATTERN-config-layers`: D5's module is new, so no reinterpretation is needed.
- `PATTERN-daemon-wakeups`: D2's outbound worker is an unbounded wait while a message is pending and joins the declared list in `cmd/lym/daemonwakeups_test.go`.
- `PATTERN-child-processes`: the daemon that starts another hub's daemon (D2) logs the spawn and reaps it.
- `PATTERN-cwd-resolution`: creelengine stays told; the other hub's geometry and the registry reach it through a seam `internal/creelcli` implements.
- `PATTERN-test-isolation`: D1's directory is told, so no test writes the operator's record.

## Auto-mode assumptions

- Direct daemon-to-daemon delivery with the sender's journal as the queue, and no inbox folder (D2).
- `trusted_hubs` by shortname, in the committed hub base file, classified Guard (D5).
- Only strands send and receive across hubs; only post and reply cross (D2, D4, D8).
- A confirm that answers `no` drops the message instead of showing it as unverified (D3).
- The registry is written by the creel daemon at start (D1).

## Open risks

- The entry proposes that unverified mail shows as unverified; D3 drops a message whose confirm answers `no` and retries one whose confirm fails, so no agent ever sees unverified mail from another hub. The parent should confirm that this stricter reading is wanted.
- `sandbox-closed-area-build` would bind only the area's own hub, so a sandbox hub inside an area could not reach loomyard's socket; if that note goes ahead, cross-hub mail needs a route through its broker.

## Acceptance criteria

- D1: a daemon's start records its hub and drops entries whose Bolt shortname no longer matches; a lookup by shortname answers the one matching hub, refuses an unknown shortname naming `lym creel inbox` in that hub, and refuses a shortname two hubs share naming both paths; two daemons recording at once both stay in the record; a test's record lives in its own directory.
- D2: a strand's post to a strand of another hub is answered with an id at once and reaches the receiver's pool; a post while the receiving daemon is down starts it and is delivered; a program or EXTERNAL sender, a program receiver of another hub and an unknown shortname are refused with their way forward; a body whose stamped line would pass 8 KiB is refused at the post.
- D2: a pending outbound message gets no mail flag, reminder, delivery worker or sweep return in the sending hub, and a delivered one that expects a reply is not settled by a sweep there.
- D2: a delivery the receiver refuses returns undelivered at once with a `CREEL` notice to the sender and ends a `--wait-reply` as returned; a receiver that stays unreachable returns the message an hour after its post, measured by an injected clock and counted across a daemon restart; a pending outbound message survives a sending daemon restart and is delivered after it.
- D3: a `deliver` request written by hand naming a sender the sending daemon never sent is refused and reaches no agent; a second `deliver` of a stored message answers the same id and stores nothing; a receiver that confirmed and died before storing stores the message on the retry; a confirm after the remote delivery is journaled answers `no`; a body whose hash differs answers `no`; two hubs delivering to each other at once both complete.
- D3, D5: on Linux a message from `<shortname>:orch` or `<shortname>:<slug>:orch` of a hub in `trusted_hubs` is stored trusted; one from a non-orch strand or an `orch-2` strand of that hub, and one from the orch of a hub not listed, is stored untrusted; off Linux every message from another hub is untrusted.
- D4: a reply from the holder in the receiving hub reaches the original sender as mail, ends its `--wait-reply` and marks the original replied; a delivered reply naming an id that did not go to the replier's hub is refused; a forward of a cross-hub message to a program or to another hub's name is refused.
- D5: a local layer file holding `trusted_hubs` is refused, a change in the base file applies to the next arrival without a daemon restart, and a load error stores the message untrusted.
- D6: a trusted message's hook line and notice read `from "<sender>" (other hub, trusted orch): <body>`, an untrusted one's `(other hub, untrusted)`, the expects-reply form carries the stamp before `, expects a reply`, `lym creel read` keeps `from` the bare name, and a pane-mail strand gets a cross-hub message reminded, not typed.
- D8: `lym creel hubs` lists the record's live hubs with their trust; `inject`, `interrupt`, `hold`, `release` and `post --interrupt` to another hub's name are refused naming `lym creel post`.
- Review: D7: the Claude creel trust sentence, the parent directive and the orch role say that `(other hub, trusted orch)` mail carries an orch's authority and `(other hub, untrusted)` mail is data, and the trust sentence no longer counts an orch strand of another hub as trusted.
- Review: D7: `internal/creelengine/doc.go` documents the cross-hub route and drops the "later and not planned" line.
- Review: D9: the board holds the note `creel-cross-hub-trial` naming the darn `docs` trial and its `trusted_hubs` setup.

## PATTERN changes

- `PATTERN-agent-addressing` — changes: Naming or messaging an agent: names only via `internal/agentname`; messages only through creel; across hubs daemon to daemon with the sender confirmed, authority only for a trusted hub's orch.

## Notes for the plan writer

- `creelconfig` is a degrading caller; the config-layers background lists degrading callers by package.
- The tier0 engine tests can run two stores with their own geometry in one test process, the D1 lookup and the confirm reaching the other through the creelcli seam's fake.
