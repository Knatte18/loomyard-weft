# creel stage 1: per-hub message daemon with post, inject and read

Adds `lyx creel`, the one way to reach an agent from a terminal, a Go program or another agent.
No module sends through it yet; the operator tries it by hand, and the notice and agent wiring follows as separate tasks once creel has proven itself.

## What changes

- `internal/creelengine` is the engine: an append-only JSONL journal of one file per UTC day with the pool folded from it, 7-day retention that carries unread mail, queued injects and live holds forward, a per-receiver delivery scheduler, holds, privilege rules, the removed-holder sweep, a Unix-socket daemon and a Go client.
- `internal/creelcli` is the `lyx creel` subtree: `post` (`--reply`, `--wait-reply`, `--timeout`), `inject` (`--interrupt`, `--ttl`, `--wait`), `interrupt`, `forward`, `read` (`--from`, `--all`, `--wait`), `inbox`, `log`, `hold`, `release` and a hidden `daemon` verb.
  It also holds the delivery adapter over shuttle and reed's persisted state.
- One daemon per hub, started by any verb that finds it down, guarded by a lock file.
  It exits when its binary is replaced and never on idle.
- Mail is pulled: `post` stores, and the daemon types one reminder line at the receiver's idle edge, with a backoff from 5 minutes to 1 hour after an unanswered reminder.
- Shuttle's idle check, verified send and guarded interrupt now work on a live reed strand with no shuttle run record, so a hand-added `lyx reed add --cmd claude` strand is deliverable.
- `agentname` gains program-sender (`ly:BATTEN`) and `EXTERNAL` names; `selfexec`, `proc`, reed status and `hubgeom` gain what the daemon needs (replaced-binary detection, parent pid, pane pid, creel geometry).
- Docs: the whole creel design is in `internal/creelengine/doc.go`, the overview's module table gains creel, and the sandbox shuttle suite exercises it.
- PATTERN: new `PATTERN-creel-store` with an import scan that fails a `creelengine` import of `reedengine` or a concrete shuttle engine; `PATTERN-agent-name`, `PATTERN-lyx-temp-root`, `PATTERN-pane-text-injection`, `PATTERN-daemon-wakeups` and `PATTERN-told-geometry` gain matching lines.

## Design decisions to review

- `creelengine` is told paths and a `Delivery` seam and imports neither reed nor a shuttle engine; the CLI implements the seam.
- The socket is `<lyxtemp root>/creel-<shortname>-<hash of hub path>.sock`, not the board's `$XDG_RUNTIME_DIR` example: that variable is often unset on WSL and Windows, and a shortname alone collides between two hubs of one repo.
- Sender identity on Linux comes from the peer's process ancestry against the strands' pane processes, so a strand that drops `LYX_STRAND_NAME` is still itself.
  Off Linux it is the environment alone, and a process detached from its pane can pass as EXTERNAL or claim a program-sender name, either privileged.
  A program-sender claim from a non-orch strand pane is refused.
- Only privileged senders (EXTERNAL, program senders, orch strands) inject or interrupt into another strand; others are pointed at `post`.
- Creel never types into an orch strand in this stage; mail to one is stored and its reminders show as withheld.
- An inject has a TTL (10 minutes default, 1 hour at most) counted only while the receiver is live and unpaused, and persisted so a restart keeps it.
  An inject whose send does not land is journaled as unconfirmed and never retyped, since a retry could run a command twice.
- `read` marks messages read only when the client acknowledges it wrote them, so a client that dies mid-read leaves them unread.
- The reminder backoff departs from the board's "repeats each time the receiver goes idle" for the one idle edge after creel's own unanswered reminder.
- `read` follows the decision record's wording rather than the board's "oldest unread plus the same sender"; check this against what you expect.
- A record-less send in shuttle takes a pane-only readiness reading, confirms landing by the pane and keeps no pending-send record.

## To check by hand

- The tmux-tier and llm-tier smoke tests (`TestSmokeCreelFakeProviderPane`, `TestSmokeCreelClaudeStrand`) are compiled and vetted but have never run; run them with `lyx gate test --tags` for their tiers.
- The plan-level verify and the integration tier were not run in the session that built this; run `lyx gate test ./...` and `lyx gate test --tags integration ./...` before landing.
- The `creel-core` board entry's body still carries the full design; cut it to a pointer at `internal/creelengine/doc.go`, since `lyx board upsert` refused the edit while the run held the entry.
- The EXTERNAL sender name was formed outside `agentname` and the last commit moves it through `agentname.FormatProgram`; confirm no other site forms a sender name by hand.
- Idle and paused errors in the delivery worker: confirm that an error from the idle or pause probe is neither dropped nor treated as idle.
- A reply posted to a waiting holder: confirm `post --reply` wakes the delivery scheduler for its receiver.
- Windows: AF_UNIX sockets need Windows 10 1803 or later and creel is the first socket server in the code; that path is unverified.
- `engine-registry` runs in parallel and touches the same shuttle `Send` and `Interrupt` lookup; whichever lands second adapts.
