# Decision record — creel stage 1

## Goal

Build creel stage 1: the per-hub creel daemon, its message pool and journal, every verb of the board design, and delivery into agent panes through shuttle, gated on the receiver being idle.
No other module sends through creel when this lands; the operator tries it from a terminal, and stages 2 and 3 (`creel-notices`, `creel-agents`) wire modules to it later on the operator's word.

## Scope

In:

- A new engine package `internal/creelengine` (pool, journal, retention, delivery scheduler, holds, privilege, removed-holder sweep, daemon server, socket protocol, Go client) and a CLI package `internal/creelcli` (the cobra subtree, hub wiring, and the delivery adapter over shuttle and reed's persisted state).
- Verbs: `post` (with `--reply`, `--wait-reply`), `inject` (with `--interrupt`, `--ttl`, `--wait`), `interrupt`, `forward`, `read` (with `--from`, `--all`, `--wait`, `--timeout`), `inbox`, `log`, `hold`, `release`, and a hidden `daemon` verb.
- Shuttle: idle check, verified send and guarded interrupt on a live reed strand with no shuttle run record (D17).
- `internal/agentname`: the non-strand sender names (D4).
- `internal/lyxdirs`: the creel directory name. The socket uses the existing `lyxtemp` root accessor.
- Docs, PATTERN, sandbox coverage and test-budget rows (D20).

Out:

- Any module sending through creel: the orch watcher's and batten's notices and injects (`creel-notices`), the parent-review gate, drivers' stop reports and moving `lyx shuttle send`/`interrupt` behind creel (`creel-agents`).
- Typing into an orch strand (D16); `PATTERN-orch-pane-single-writer` stays as written until `creel-notices`.
- Cross-hub sending, an MCP layer, a `creel.yaml` config module, and the shuttle engine registry (`engine-registry`).

## Decisions

### D1 — Two packages, a delivery seam, layering creel → shuttle → reed

**Decision:** `internal/creelengine` is told the hub's paths and a `Delivery` seam, and derives nothing (`PATTERN-told-geometry`).
The seam lists the hub's strands (name, worktree, live or dormant, pane process), says whether a strand is idle and whether a pause covers it, types a line into a strand, and sends the guarded interrupt.
`internal/creelcli` implements it: strands from each hub worktree's persisted reed state, read-only; idle, send and interrupt through a shuttle `Runner` per worktree; pause coverage through `pauserecord`.
Each worktree's geometry comes through `hubgeom` and the `cliwire` constructors, never resolved in creel (`PATTERN-cwd-resolution`).
`creelengine` imports neither `reedengine` nor a concrete shuttle engine, and names no tmux command.
**Rationale:** the board's layering (creel decides what and when, shuttle knows the LLM, reed does keystrokes) becomes an import rule, and the fake seam makes the scheduler testable without panes.

### D2 — The daemon: one per hub, started on demand

**Decision:** `lyx creel daemon` (hidden) runs the per-hub daemon from the hub directory.
One instance per hub is held by a lock file under `<hub>/_board/.lyx/creel/`; a second daemon that cannot take the lock exits.
Any creel verb that cannot connect spawns it detached, logs the spawn (`PATTERN-spawn-observability`), and retries the connect a capped number of times; under `go test` it never spawns, by the same suppress argument `reedengine.SpawnWatchdog` takes.
The daemon checks its binary's identity from its own poll and, when the binary is replaced, closes its listener, releases the lock and exits; the next verb starts the new binary's daemon.
It never exits on idle.
On start it rebuilds the pool from the journal (D6), including queued injects with their remaining TTL.
In standalone geometry every creel verb refuses, naming that creel needs a hub.
Creel's daemon stays apart from reed's watchdog: reed never imports creel.
**Rationale:** the board asks for a daemon of creel's own started like reed's server.
An exit rather than a self re-exec keeps the lock and socket handover in one place, the start path, since an exec drops a close-on-exec lock and opens a window for a second daemon.

### D3 — Socket and protocol

**Decision:** the socket is `<lyxtemp root>/creel-<shortname>-<h>.sock`, where `<h>` is the first 8 hex digits of a SHA-256 of the hub's absolute path.
This is a new use of the lyxtemp root, a long-lived file beside the run directories, and the `PATTERN-lyx-temp-root` background gains a line for it in the same commit; `Root.Sweep` touches no unprefixed entry.
The daemon that holds the lock removes a socket file no listener answers before it binds.
Each connection carries one JSON-lines request and its response; a wait verb keeps its connection open until its answer.
Every response carries the hub path, and a client refuses a daemon answering for another hub.
A request that changes the pool is acknowledged only after its journal line is appended and synced.
**Rationale:** this departs from the board's example path `$XDG_RUNTIME_DIR/lyx/creel-<shortname>.sock` on two points.
`XDG_RUNTIME_DIR` is unset on Windows and often under WSL, while the lyxtemp root is per user, 0700-checked and short enough for `sun_path`.
The shortname alone collides between two hubs of one repo on one machine, sandbox hubs included; so the board's premise that an address alone picks a hub's socket no longer holds, and a cross-hub feature finds it by shortname and hub-path check.

### D4 — Sender identity

**Decision:** on Linux the daemon reads the connecting peer's process id and walks its ancestry against the strands' pane processes; a peer inside a strand pane is that strand, and a `$LYX_STRAND_NAME` that disagrees with it is refused.
Elsewhere, and for a peer in no strand pane, the sender is `$LYX_STRAND_NAME` when it parses as an agent name of this hub's shortname.
So a fork, a Bash call or any child process inside a strand signs as its strand; a verify run, whose environment `verifytree` strips, still resolves to its strand on Linux.
A Go program names itself through the client API as `<shortname>:<NAME>` with an upper-case name (a program sender), such as `ly:BATTEN`; this supersedes the board's lower-case `ly:batten` example.
The CLI has no flag for the sender.
Anything else is `<shortname>:EXTERNAL`, and the envelope adds what creel sees: the terminal, the parent process's name and the working directory, shown as `from ly:EXTERNAL (claude, pts/4, ~/Code/…)`.
`internal/agentname` forms, parses and tells apart these names, and the `PATTERN-agent-name` background gains a bullet beside its grammar line for the upper-case marker form; strand names are still formed once by reed.
Bound: off Linux, identity is the environment alone, so an agent that unsets `LYX_STRAND_NAME` passes as EXTERNAL and gains D9's privilege.
On every platform the trust boundary is the OS user, since the socket sits in a 0700 per-user directory, and nothing guards against a hostile process of that user.
**Rationale:** the board's naming rules, made concrete; the ancestry check closes the one-command bypass where a confused agent drops its env and becomes privileged.

### D5 — Receivers and name resolution

**Decision:** a receiver is a full agent name resolved hub-wide over every worktree's reed state, or a program sender's name once that program has posted or read.
A name neither covers is refused, listing the names that exist.
`<shortname>:EXTERNAL` is never a receiver: posting or forwarding to it is refused, naming `--wait-reply` and `--reply` (D12).
A known receiver that is dead or dormant keeps its mail and queue, and delivery resumes when its strand is live.
A failed read of the worktree list or of a reed state never refuses a name it cannot see; the verb fails naming what could not be read.
**Rationale:** the board's receiver rules; the address is the role-stable name, so a respawned role receives its mail.

### D6 — Storage: one pool folded from a journal

**Decision:** creel stores everything as an append-only journal of JSONL files, one per UTC day, under `<hub>/_board/.lyx/creel/journal/`, and the pool is the fold of the retained files.
A message record carries its id, `from`, current holder, `path`, kind, subject (its first line), body, times and status.
Ids are integers that never repeat: every day file opens with a high-water record of the largest id issued, and a new id exceeds both it and the fold's maximum.
Events are posted, reminded, read, forwarded, returned, injected, unconfirmed, expired, interrupted, held and released.
The daemon is the only writer.
At each UTC day change the daemon deletes the day files older than 7 days, after appending a carried record for each unread message, queued inject and live hold they hold to today's file.
The fold is idempotent by id, so a crash between the carry and the deletion duplicates nothing.
A reader ignores unknown keys (`PATTERN-tolerant-records`), and a torn last line from a crash is skipped with one Warn.
A body is non-empty and at most 64 KiB; a longer message is a file the sender writes and the mail names.
**Rationale:** the board's single pool and 7-day retention; an event log makes the journal and the pool one source, so `log` and the fold never disagree, and no record is ever rewritten.

### D7 — Post and reminders

**Decision:** `post` stores the message for its receiver and answers its id.
For a holder with unread mail, the daemon types one reminder line through shuttle's verified send when the holder is idle: `[creel] 2 unread: #41 from ly:x:driver «subject», #42 … — run lyx creel read`, listing at most 5 messages and counting the rest.
Each subject is cut to 80 runes, has control and delimiter characters replaced by spaces, and sits inside « », marked as the sender's words, not an instruction.
The line is composed fresh from the unread set at delivery, never queued.
A reminder goes out at each idle edge (busy to idle), except the first idle edge after creel's own reminder while the unread set is unchanged: then the next reminder waits a backoff from 5 minutes, doubling to 1 hour, and a change of the unread set resets it.
This departs from the board's "repeats each time the receiver goes idle" for that one edge.
A program sender's mailbox is never reminded, since it has no pane.
Bound: an agent that never reads costs one reminder turn per backoff step at most.
**Rationale:** mail is pulled as the board decides; without the backoff, a receiver that ends its turn unread is reminded at once and loops turns.

### D8 — Read and inbox

**Decision:** `read`, `read --from <sender>` and `read --all` print the messages the board names, with envelope (id, from, path, time) and body, oldest first; with nothing unread they print that and exit 0.
The daemon reserves the messages it hands a read, so a concurrent read by the same holder skips them.
A message is marked read when the client acknowledges, on the same connection, that it wrote them to its output; a connection that closes without the ack releases the reservation and leaves them unread.
`read --wait` blocks until the caller holds unread mail, then reads as `read` does; `--timeout` bounds the wait and its expiry exits non-zero.
`inbox` prints the caller's senders with their unread counts.
`read` and `inbox` work only on the caller's own holdings; an EXTERNAL caller has none and is refused, naming `--wait-reply`.
**Rationale:** "delivered means read" holds only if a read reaches its reader; the ack closes the gap and the reservation keeps two reads from both taking one message.

### D9 — Inject and privilege

**Decision:** `inject` queues a single-line text for its receiver; a text with a control character is refused.
Per receiver, injects go out in order and ahead of reminders, each through shuttle's verified send once the receiver is idle.
An inject lives for its TTL, 10 minutes by default and at most 1 hour through `--ttl` (a larger value is refused), counted only while the receiver is live and no pause covers it; one still queued at expiry is journaled as expired and never typed.
A send shuttle reports as not landed is journaled as unconfirmed and never retyped, since a retry could run a command twice.
`--wait` blocks until the inject is typed, unconfirmed, expired or refused, prints which, and exits non-zero unless typed; it is the confirmation `creel-notices` relies on for a command inject.
Only a privileged sender injects into another strand: an EXTERNAL sender, a program sender, or any strand of role `orch`.
Any sender may inject into its own strand.
Any other sender is refused, naming `lyx creel post` as the way forward.
Bound: privilege lets those senders type arbitrary single-line text, slash commands included, into any agent's pane at its next idle; the TTL bounds how stale a typed command can be against a long busy turn, and D14 and D16 hold it from paused and orch panes.
**Rationale:** the board's inject; the TTL keeps a `/clear` queued behind a long turn from landing an hour later, while a pause or a dormant receiver still holds the queue as the board decides.

### D10 — Interrupt

**Decision:** `interrupt <receiver>` sends shuttle's guarded interrupt at once, bypassing the queue (`PATTERN-guarded-interrupt`); a receiver with no running turn gets nothing, and the verb reports it.
`inject --interrupt` sends the guarded interrupt, then queues its text at the head of the receiver's queue, so it is typed at the next idle.
Both follow D9's privilege rule, and both are refused while a pause covers the receiver.
**Rationale:** the board's "stop, and do this instead"; shuttle stays the only sender of the interrupt key, and a pause has already stopped the turn.

### D11 — Forward

**Decision:** `forward <id> <receiver>` moves an unread message the caller holds to another receiver: the holder changes, `from` stays, and the caller is appended to `path`.
A message the caller does not hold or has read, a forward to the caller itself, and an unknown receiver are refused.
**Rationale:** the board's forward, with access following the holder.

### D12 — Replies and waiting senders

**Decision:** `post --reply <id> "<text>"` addresses the original message's `from`; the caller must hold or have read that message, and an id the pool no longer holds is refused.
To a strand or program sender it is mail as any other.
To an EXTERNAL sender waiting with `post --wait-reply` it is handed over the waiter's open connection and printed; with no waiter it is journaled as undelivered.
Every waiting verb (`post --wait-reply`, `inject --wait`, `read --wait`) whose connection drops reconnects, starting the daemon if needed, and waits on the same id or holder a capped number of times.
**Rationale:** an external sender has no address, so the board's `--wait-reply` is its only way back; a reply names its message, so creel never guesses who asked.

### D13 — Holds

**Decision:** `hold <receiver> --for <duration>` (at most 1 hour; a larger value is refused) and `release <receiver>` hold and release one sender's hold on one receiver; D9's privilege rule applies.
While held, the receiver gets no injects and no reminders for mail from senders other than the holder; the holder's own injects pass, and its own unread mail is reminded alone.
Bound: holding again renews the hold with no overall cap, so a live privileged holder can mute a receiver for as long as it keeps renewing; a holder that dies mutes it for at most 1 hour.
**Rationale:** the board's hold for the watcher's context cycle, built and tested now so `creel-notices` only uses it.

### D14 — Pause

**Decision:** while a `pauserecord` entry covers a receiver (a hub entry, or the entry of the receiver's slug; a prime strand, with no slug, by a hub entry only), creel types nothing into it, injects and reminders alike, until the pause is lifted; `post` still stores.
**Rationale:** the board's 2026-10-12 decision.

### D15 — Removed holders

**Decision:** while unread mail exists, the daemon sweeps every 30 seconds: a holder is gone when it is a strand name no worktree's reed state holds on two sweeps in a row.
Each unread message it holds goes one step back along its `path` (the last forwarder, else the sender), marked with the receiver that ended unread, and is reminded there as any mail.
When that step is gone too it goes on toward the sender; when the sender is gone, EXTERNAL or a program, it is marked undelivered and stays in the journal.
A sweep whose worktree list or any reed state read fails moves nothing and counts toward no streak.
**Rationale:** the board's rule, by polling, so no strand-death event is needed; the two-sweep streak keeps a strand being replaced (removed, then re-added) from losing its mail.

### D16 — Orch strands in stage 1

**Decision:** creel never types into a strand of role `orch` in any worktree in stage 1.
Mail to an orch is stored and readable, and `inbox`, `log` and the journal show its reminders as held, naming `creel-notices`.
`inject` and `interrupt` into an orch are refused, naming `lyx orch distill`, `lyx orch refresh` and the orch's own pane.
**Rationale:** `PATTERN-orch-pane-single-writer` still gives the orch pane to the watcher, and a reminder typed mid-cycle is the very loss creel exists to prevent; stage 2 lifts this when the watcher holds through creel.

### D17 — Shuttle: strands with no run record

**Decision:** shuttle's idle check, verified send and guarded interrupt work on a live reed strand with no shuttle run record.
The idle check keeps its existing pane fallback.
The send takes the pane-only readiness reading (no hook state, no events file), confirms landing by the pane alone, never clears a text that did not land and keeps no pending-send record; a later send then finds a non-empty box and holds.
The interrupt keeps both pane guards.
This changes `PATTERN-pane-text-injection`, whose pending-send record and hook-fed "session is ready" fact a record-less send lacks; its background gains a bullet for the record-less send in the same commit.
Such a strand is driven with the runner's default engine.
`engine-registry` rewrites the same `Send` and `Interrupt` record lookup to pick the engine from the run record, and `engine-config-seams` gives strands a provider tag; a strand with neither, such as a hand-added one, keeps the default engine there too.
Bound: shuttle may now type into any live strand reed tracks in the worktree, behind the same pane guard; a strand whose pane is not an agent reads not ready and gets nothing, and a text left in the box holds later sends until someone clears it.
**Rationale:** the board requires delivery to a hand-added Claude strand, and today `Send` and `Interrupt` refuse any strand without a run record.

### D18 — Log

**Decision:** `lyx creel log` prints journal events, filtered by `--id`, `--from`, `--to` and `--since`, with ids, parties, times and subjects, never bodies.
Any caller may read it; a sender sees its own message's status through `log --id`.
**Rationale:** the board's journal view for debugging; leaving bodies out keeps "only the holder reads" true.

### D19 — The delivery loop

**Decision:** the daemon runs one worker per receiver with pending deliveries (queued injects, or unread mail not muted by D13 or D16), with at most one delivery in flight per receiver.
A worker probes idle from a 1-second base, doubling to 8 seconds while the receiver reads busy, held or paused, and returns to the base on a new post, inject or release (`PATTERN-daemon-wakeups`).
A send that shuttle holds as busy is retried at the next probe; another error is logged at Warn once per change of error.
A receiver with nothing pending is not probed.
The worker loop, the sweep, the binary-identity poll and the waiting verbs join the declared list in `cmd/lyx/daemonwakeups_test.go` and the `PATTERN-daemon-wakeups` background.
**Rationale:** a slow receiver never delays another, and pane captures back off as the wake rule requires.

### D20 — Docs, PATTERN and coverage

**Decision:** `internal/creelengine/doc.go` carries the whole creel design, the parts stages 2 and 3 build included (the note request as mail answered by `written <path>` or `defer`, the watcher keeping only its cycle logic, `PATTERN-orch-pane-single-writer` becoming "only creel decides what is delivered into an agent's pane", drivers' stop reports by post), and the `creel-core` board body is cut to a pointer at it in the same task.
`docs/overview.md`'s module table gains creel.
A new PATTERN entry, `PATTERN-creel-store`: the creel journal is written only by the creel daemon in `internal/creelengine`, and `internal/creelengine` reaches a pane only through its delivery seam.
An import scan checks the second half; the first holds because the journal writer is unexported and the daemon is its only caller, and review checks it.
The sandbox suite exercises creel (`PATTERN-sandbox-coverage`).
**Rationale:** CLAUDE.md's docs-in-the-same-commit rule and `PATTERN-documentation-lifecycle`; the stage-2 and stage-3 tasks read the design from `doc.go`.

## Constraints

- `PATTERN-cli-cobra`: one cobra subtree under the root.
- `PATTERN-test-speed`, `PATTERN-test-isolation`, `PATTERN-testkit`: injected clocks, tmux-tier and llm-tier tests through `tmuxkit.Main`, the existing `shuttlefake` kit for fake panes.
- `PATTERN-log-levels`: the daemon's per-tick logging.
- CLAUDE.md: agents run in interactive tmux; `cmd/lyx/testdata/test-budget.yaml` gains rows for the new packages.

## Auto-mode assumptions

- No `creel.yaml`: retention, TTLs, backoffs and caps are constants in `creelengine`.
- Socket name carries a hub-path hash (D3).
- Sender identity from process ancestry on Linux (D4).
- Privileged senders are EXTERNAL, program senders and orch strands (D9).
- A reminder backs off after its own unanswered turn (D7).
- Injects carry a TTL counted while live and unpaused, and an unconfirmed inject is never retyped (D9).
- Holds carry an expiry (D13).
- Orch strands are never typed into in stage 1 (D16).
- `read` reserves and marks read on the client's ack (D8).
- The daemon exits on a replaced binary rather than re-exec (D2).
- Program senders' mailboxes exist from first use (D5).

## Open risks

- `engine-registry` runs in parallel and touches the same shuttle `Send`/`Interrupt` lookup (D17) and the hardwired `claudeengine` construction sites, to which creel's adapter adds one; whichever task lands second adapts to the other.
- AF_UNIX sockets on native Windows need Windows 10 1803 or later, and creel is the first socket server in the code; the Windows path is not verified here.
- Off Linux a confused agent can pass as EXTERNAL (D4).
- A subject typed into an agent pane is sender-written text; the delimiting in D7 lowers the prompt-injection risk but does not remove it.
- Creel stays unproven against an orch until `creel-notices`, since stage 1 never types into one (D16).
- Reading every worktree's reed state each sweep grows with the hub's pairs; reads are read-only and run only while unread mail exists.

## Acceptance criteria

- A post to a live idle fake strand is stored, acknowledged with its id, and reminded once; a busy receiver is reminded at its next idle edge; a reminder turn ending unread backs off and a new message resets the backoff.
- `read`, `read --from`, `read --all` and `inbox` return the sets the board names; `read` with nothing unread exits 0; `read --wait --timeout` expiring exits non-zero.
- A read whose client dies before acking leaves the messages unread; two concurrent reads by one holder never return the same message.
- An unknown receiver is refused, listing the names that exist; posting or forwarding to EXTERNAL is refused naming `--wait-reply`; an empty or over-64-KiB body is refused.
- A dead or dormant receiver keeps its mail and queue, and delivery resumes when its strand is live.
- A removed holder's unread mail goes one step back along its path after two sweeps, then on toward the sender, and is marked undelivered when no step is left; a strand removed and re-added between sweeps keeps its mail; a failed reed-state read moves nothing.
- `forward` changes holder and path and keeps `from`; forwarding a message the caller does not hold or has read, to itself or to an unknown name, is refused.
- An inject is typed before reminders, in queue order; an expired inject is never typed, and its TTL does not run while the receiver is paused or dormant; an unlanded inject is journaled unconfirmed and never retyped; `--wait` reports each outcome with the matching exit code.
- `--ttl` or `hold --for` above 1 hour is refused.
- A non-privileged sender's inject or interrupt into another strand is refused naming `lyx creel post`; a self-inject passes; on Linux, a strand process with `LYX_STRAND_NAME` unset is still its strand and not EXTERNAL.
- `interrupt` sends nothing into a pane with no running turn and is refused under a pause; `inject --interrupt` interrupts and then types the text at the next idle.
- A hold mutes other senders' reminders and injects and lets the holder's own pass, and ends at its expiry.
- A pause covering the receiver types nothing until it is lifted; post still stores.
- Nothing is typed into an orch strand of any worktree; inject and interrupt into it are refused with their way forward.
- `post --reply` reaches a waiting EXTERNAL sender; with no waiter it is journaled undelivered; a reply to an id the pool no longer holds is refused.
- A daemon killed mid-run restarts on the next verb and rebuilds the pool from the journal, queued injects included; ids never repeat, also after retention deleted the files holding the largest ids; a torn last line is skipped; each waiting verb reconnects and keeps waiting.
- A daemon whose binary is replaced exits and the next verb starts one daemon on the new binary.
- Retention deletes UTC day files older than 7 days and keeps unread mail, queued injects and live holds; a crash between carry and deletion duplicates nothing.
- Two concurrent first verbs start one daemon; a client refuses a daemon answering for another hub.
- Shuttle's idle check, send and guarded interrupt work on a live strand with no run record; a send that does not land leaves its text and the next send holds.
- In the tmux tier, a post, reminder and inject reach a fake-provider pane end to end through the real daemon and socket.
- In the llm tier, on a hand-added `lyx reed add --cmd claude` strand, the idle check reads it right, and a reminder and an inject are typed at idle.
- The PATTERN scan fails a `creelengine` import of `reedengine` or a concrete shuttle engine.

## Notes for the plan writer

- `Runner.Send`, `Runner.Interrupt` and `Runner.Inject` in `internal/shuttleengine/run.go` refuse a strand without a run record through `FindRun`; `Runner.SessionIdle` in `session.go` already falls back to the pane.
- `reedengine.SpawnWatchdog` is the model for D2's detached spawn and its test suppression; `fabricengine.HubScratchDir` gives `<hub>/_board/.lyx`; `selfexec` holds the binary-identity check.
- No code under `internal/` listens on or dials a socket today, so D3's transport has nothing to copy.
- `orchengine.QueueNotice` (`internal/orchengine/notice.go`) is the queue stage 2 folds into creel.
- `internal/testkit/shuttlefake` fakes `Engine` and `ReedOps` for the adapter's tests; reed's `proctree` reads process ancestry.
