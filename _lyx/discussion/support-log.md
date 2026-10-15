# Support log — creel stage 1

## Interview

Autonomous mode: no operator answered; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

Read the board entry `creel-core` and the related entries `creel-notices`, `creel-agents`, `engine-registry`, `codex-engine`, `reed-slack-relay` and `pause-background-work`.
Read the shuttle send, interrupt and idle paths, reed's package doc, name directory and watchdog spawn, `agentname`, `pauserecord`, `lyxtemp`, `selfexec`, and the PATTERN backgrounds for temp root, daemon wakeups, tolerant records, agent name, pane text injection, guarded interrupt, provider seam and spawn observability.

Findings that shaped the design:

- `Runner.Send`, `Interrupt`, `Inject` and `PaneRunningTurn` refuse a strand with no shuttle run record; `SessionIdle` falls back to the pane. The board's hand-added Claude strand therefore needs a shuttle change (D17).
- Shuttle run records live per worktree and reed state is per worktree on one hub-wide tmux server, so the hub daemon builds one shuttle runner per worktree (D1).
- `XDG_RUNTIME_DIR` appears nowhere in the code; the lyxtemp root is the existing per-user, 0700-checked directory, and tmux sockets already obey `sun_path` there (D3).
- Hub-level ephemeral state lives under `<hub>/_board/.lyx/` (pause, hibernate), so creel's goes there too (D2, D6).
- `PATTERN-orch-pane-single-writer` gives the orch pane to the watcher until stage 2 rewrites it (D16).

### Turn 2 — design batch, self-answered

1. Where the socket lives, and how two hubs with one shortname stay apart. Recommended and picked: lyxtemp root plus a hub-path hash (D3).
2. Whether creel types into the orch in stage 1. Recommended and picked: never (D16).
3. Who may inject. Recommended and picked: EXTERNAL, program senders, the orch, and self (D9).
4. How reminders avoid a turn loop. Recommended and picked: backoff after creel's own unanswered turn (D7).
5. How a hand-added strand gets delivery. Recommended and picked: shuttle's record-less path (D17).
6. How storage meets 7-day retention with unread mail kept. Recommended and picked: day-file journal folded into the pool, unread carried forward (D6).
7. Whether a `creel.yaml` is needed. Recommended and picked: no (Auto-mode assumptions).

### Turn 3 — lens review

Every lens fork returned a report.

- goal-scope: inject TTL contradicts the board's pause and dormant hold — fixed: TTL counts only live, unpaused time (D9, D14).
- goal-scope: reminder backoff departs from the board unnamed — fixed: D7 names the departure.
- goal-scope: no real hand-added Claude test — fixed: llm-tier criterion added.
- goal-scope: `doc.go` gets only this record's design — fixed: D20 carries the whole design and cuts the board body to a pointer.
- goal-scope: orch unproven until stage 2 — fixed: Open risks line.
- goal-scope: `inject --wait` not tied to stage 2 — fixed: D9 names it.
- goal-scope: right thing to build — no change needed.
- leanness: D3's cross-hub clause — partly rejected: kept as one clause, since board-fit showed it supersedes a board premise; shortened.
- leanness: Constraints repeat decisions — fixed: only constraints no decision carries remain.
- leanness: `doc.go` stated thrice, `lyxdirs` twice — fixed: Goal and Scope shortened, D20 holds it.
- leanness: QueueNotice clause repeats Scope — fixed.
- leanness: Auto-mode lines repeat decisions — rejected: required section, lines are pointers.
- leanness, pattern-fit: "four things" tally — fixed: D1 drops the count.
- leanness, pattern-fit: import scan cannot see journal writes — fixed: D20 says the write rule holds by an unexported writer and review; criterion narrowed.
- board-fit, premises: one-orch premise, `orch-any-worktree` — fixed: D9 and D16 cover every strand of role `orch`.
- board-fit: overlap with `engine-registry` in `Send`/`Interrupt` — fixed: D17 and Open risks.
- board-fit: per-strand engine tag belongs to `engine-config-seams` — fixed: D17.
- board-fit: `ly:batten` example vs upper-case senders — fixed: D4 says it supersedes it.
- board-fit: D3 vs "costs nothing" premise — fixed: D3 says it no longer holds.
- board-fit: `spawn-audit-log` — rejected: a note, no planned work to fit.
- board-fit: dangling `run-message-routing` reference — rejected: in the board body, not the record; for the operator.
- premises: `lyxtemp` already has a root accessor — fixed: Scope says use.
- premises: no socket server precedent — fixed: plan-writer note.
- premises: child processes carry `LYX_STRAND_NAME`, verify strips it — fixed: D4.
- premises: prime coverage by hub entry only, unconfirmed — fixed: D14 states it; Plan confirms the call.
- scenarios: `env -u` turns an agent EXTERNAL and privileged — fixed: D4 resolves the sender from process ancestry on Linux; bound and risk for other platforms.
- scenarios: ids reused after retention — fixed: high-water record (D6).
- scenarios: unlanded send disposition — fixed: unconfirmed, never retyped (D9); D17 bound names the held box.
- scenarios: concurrent reads — fixed: reservation (D8).
- scenarios: restart of queued injects and waiters — fixed: D2 and D12.
- scenarios: lock dropped across a re-exec — fixed: the daemon exits on a replaced binary (D2).
- scenarios: replaced strand looks gone — fixed: two-sweep streak (D15).
- scenarios: carry-forward crash duplicates — fixed: fold idempotent by id (D6).
- scenarios: unbounded hold renewal — fixed: bound stated (D13).
- scenarios: interrupt under a pause — fixed: refused (D10).
- scenarios: empty-state and invalid-input cases — fixed: criteria added.
- scenarios: day boundary timezone — fixed: UTC (D6).
- pattern-fit: D17 changes `PATTERN-pane-text-injection` unnamed — fixed: D17 names it and its background gains a bullet.
- pattern-fit: daemon loops not on the wakeups list — fixed: D19.
- pattern-fit: geometry through `hubgeom`/`cliwire` — fixed: D1.
- pattern-fit: new use of the lyxtemp root — fixed: D3, background line.
- pattern-fit: agentname background grammar — fixed: D4.
- pattern-fit: board body trimmed at landing — fixed: D20.

## Rejected alternatives

- Socket at `$XDG_RUNTIME_DIR/lyx/creel-<shortname>.sock`: unset on Windows and often in WSL, and the shortname alone collides between hubs of one repo.
- Shortname-only socket with a refusal on collision: sandbox hubs would refuse each other.
- Amending `PATTERN-orch-pane-single-writer` in stage 1 so creel types into the orch: reminders mid-cycle reproduce the loss creel exists to fix, and no watcher hold exists yet.
- Refusing the orch as a receiver outright: storage and read of orch mail are worth testing now, and a held reminder shows the state honestly.
- Reminding at every idle edge with no backoff: an agent that ends its turn unread is reminded at once, looping turns.
- A mailbox file per receiver: the board chose one pool; forward and the sweep become holder changes.
- Rewriting a pool snapshot through `internal/state`: two sources of truth beside the journal; the fold keeps one.
- Marking read when the daemon writes the response: a client killed before printing would lose mail marked read.
- A `--as <name>` CLI flag for senders: any agent could claim any name with no env change.
- Injects with no TTL: a `/clear` queued behind a long turn would land into a session that moved on.
- A TTL in wall time, expiring under a pause: contradicts the board's hold until the pause is lifted.
- Retyping an unlanded inject: a command could run twice.
- Self re-exec of the daemon through `selfexec`: the exec drops a close-on-exec lock, opening a window for a second daemon.
- Sender identity from the environment alone on Linux: an agent that unsets it becomes EXTERNAL and privileged.
- A prime-only orch exclusion: a pair's orch would be typed into while its watcher owns the pane.
- A `creel.yaml` config module: no tuning was asked for, and constants keep the surface small.
- Creel's own pane probe or tmux calls: the board puts idle and keystrokes in shuttle and reed.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 socket location and collision — resolved, auto-pick: lyxtemp root with hub-path hash (D3).
- Q2 orch in stage 1 — resolved, auto-pick: never typed into (D16).
- Q3 inject privilege — resolved, auto-pick: EXTERNAL, program senders, orch, self (D9).
- Q4 reminder loop — resolved, auto-pick: backoff 5 minutes doubling to 1 hour (D7).
- Q5 hand-added strands — resolved, auto-pick: shuttle record-less path (D17).
- Q6 storage and retention — resolved, auto-pick: day-file journal, fold, carry-forward (D6).
- Q7 config module — resolved, auto-pick: none.
- Q8 inject TTL — resolved, auto-pick: 10 minutes default, 1 hour max, wall time (D9, D14).
- Q9 hold expiry — resolved, auto-pick: required duration, at most 1 hour (D13).
- Q10 read marking — resolved, auto-pick: on client ack (D8).
- Q11 program mailboxes — resolved, auto-pick: exist from first use, never reminded (D5, D7).
- Q12 reply to an EXTERNAL sender — resolved, auto-pick: `post --reply <id>`, handed to a `--wait-reply` waiter (D12).
- Q13 sender identity bypass — resolved, auto-pick: process ancestry on Linux, env elsewhere (D4).
- Q14 daemon on a replaced binary — resolved, auto-pick: exit, next verb restarts (D2).
- Q15 unlanded inject — resolved, auto-pick: unconfirmed, never retyped (D9).
- Q16 which orch is excluded — resolved, auto-pick: every strand of role `orch` (D16).
